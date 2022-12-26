# Analysis of Naver Map Application's Encrypted Database

고려대학교 정보보안학과 석사과정 2022020994 이도현

## Introduction

##### Research Questions

한국의 안드로이드 OS 점유율은 70% 이상으로 대부분의 국민이 안드로이드를 사용하고 있다. 안드로이드에서 사용되는 지도 어플리케이션 중 "네이버 지도" 어플리케이션은 플레이스토어 기준 다운로드 횟수가 5천만에 이르며, 유사 어플리케이션들에 비해 압도적인 점유율을 가지고 있다. 네이버 지도에서는 사용자의 위치, 경로, 관심사, 예약 정보등이 사용되어 이러한 정보들이 남는다면 수사에 중요한 정보로 활용될 가능성이 높다. 네이버 지도 어플리케이션을 분석하는 과정에서 사용자의 활동기록이 저장된 데이터베이스가 암호화된 것을 확인하였으며, 이를 복호화하는 방법에 관하여 논하였다.

##### Summary of Research

안드로이드 에뮬레이터인 "녹스 앱플레이어"를 사용하여 어플리케이션과 데이터베이스를 추출하였고, JEB 디컴파일러 및 IDA를 사용하여 암호화 부분에 관한 정적 분석을 시행하였다. 정적 분석 결과 안드로이드 키스토어의 키값을 받아오는 것을 확인하여 해당 키값을 얻는데 실패하였고 fridump3을 통한 메모리 덤프로 sqlcipher가 데이터베이스를 복호화하는 키값을 얻으려 하였으나 덤프된 메모리 중 키값을 얻는데 실패하였다.

##### Naver Map Database

네이버 지도는 SQLite databse를 사용하며, /data/data/com.nhn.android.nmap(패키지명)/database 경로에 존재하는 데이터베이스는 bookmark.db, google_app_measurement_local.db, route-history.db, search-history.db, webviewCache.db로 총 5개이다. 이 중 암호화 되어있으며 사용자의 활동 기록이 담긴 것으로 추정되는 것은 bookmark.db, route-historu.db, search-history.db 3개이다.



## Background

##### Android KeyStore

Android Keystore는 어플리케이션 및 시스템이 사용하는 key 값들을 안전하게 보관해주는 시스템이다.

<img src="file:///C:/Users/dlehg/AppData/Roaming/marktext/images/2022-12-26-20-50-13-image.png" title="" alt="" width="379">

                        <KeyStore 구조>

키스토어의 마스터키는 /dev/urandom을 통해서 생성되며, AES 128바이트로 암호화되어 /data/misc/keystore/user_0/.masterkey로 저장된다.

AES 암호화키는 password와 salt값을 이용해서 (KEK, PBKDF2WithHmacSHA1) 해시를 통해 생성된다.

<masterkey version 2 기준>

.masterkey의 첫 4바이트는 버전, 키종류, flag, info이고 5~20바이트는 IV값, 21~68바이트는 암호화된 마스터키 구조체, 69~84바이트는 salt값이다.

password와 salt값을 통해 마스터키 구조체를 복호화한 후, 복호화된 마스터키 구조체의 복호화키값으로 개인키를 풀 수 있다.

참고: [Android Keystore 관련 정보](https://cozyu.tistory.com/329)



## Literature Review

[안드로이드 키스토어의 무결성과 기밀성에 관한 연구] Sabt, M., Traorè, J. (2016). Breaking into the KeyStore: A Practical Forgery Attack Against Android KeyStore. Computer Security – ESORICS 2016. Lecture Notes in Computer Science(), vol 9879. Springer, Cham.

[안드로이드 키스토어 아키텍쳐에 관한 연구] J. H. Park, S. -M. Yoo, I. S. Kim and D. H. Lee, "Security Architecture for a Secure Database on Android," in *IEEE Access*, vol. 6, pp. 11482-11501, 2018, doi: 10.1109/ACCESS.2018.2799384.

[SNS 어플리케이션의 db 복호화에 관한 연구] S. Shin, S. Kang, G. Kim, and J. Kim, “Study on SNS Application Data Decryption and Artifact,” *Journal of the Korea Institute of Information Security & Cryptology*, vol. 30, no. 4, pp. 583–592, Aug. 2020.



## Methods

##### database 파일 분석

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-15-22-image.png)

네이버지도 어플리케이션에서 추출한 5개의 데이터베이스이다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-16-21-image.png)webviewCache.db의 경우 SQLite format 3. 이라는 시그니처를 확인할 수 있다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-17-40-image.png)

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-18-22-image.png)

bookmark.db, route-history.db, search-history.db의 경우 sqlite의 시그니처를 확인할 수 없었고, 시작과 끝에서 일정한 형식의 헤더를 찾을 수 없었다.

##### 정적분석

안드로이드 디컴파일러인 JEB 4.18을 사용하여 분석했다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-14-24-image.png)

데이터베이스 이름인 search-history.db를 검색하여 db가 사용되는 위치를 찾았다. 암호화된 3개의 파일 중 bookmark.db를 제외한 두개의 파일명이 모여있었다.

e.a는 d에 context0, 디비명, passphrase를 넣는다.



![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-40-50-image.png)

d에서 디비명은 s, passphrase는 s1으로 바뀌고, super를 통해 상위 클래스를 호출한다. d의 상위클래스는  a이다.(public final class d extends a)

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-48-02-image.png)

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-48-55-image.png)

a에서 passphrase s1은 Y로 바뀌고, Y는 다시 c.a.a로 들어간다.



![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-49-29-image.png)

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-49-56-image.png)

e a에서 Y는 s1으로 바뀌고,  s1에 해당하는 키값이 s2에 들어간다.

sharedPreferences0은 /data/data/com.nhn.android.nmap/shared_prefs/com.nhn.android.nmap_preferences.xml에 저장된다.

    <string name="ROUTE_HISTORY_CACHE_PASSPHRASE">M5JV52QVuVRJuiPQ4/4vztVBMi0T+tlpbhAFzoEuxovL2FF+fcdv/Sgd2fxF7jllcPK8DK+Pk8FX&#10;3y0DC85SXuT2ZXv72mlO3xk8Rb6MuSGVyHuweMwopqCGbJNq7zEMxLzfFz3z4hLwe8kb/yPXKb1o&#10;Ae36WTUt5XDOs/Rvbal+sDE2i2PStDqLISlXOfAhit+y04D19NazX9AYBnVwLC3zUwGYtOu+OoeM&#10;TKKpHtPeTTB/ob3W0eT/9CO20zKkRASrMEpC3P5bVHloS0p8rsh+qpx7wjn1gauCAXqOivcD9ZBa&#10;veofgc7kljK9mhYJ4W6L4yxKUOG1HQXEOdODWQ==&#10;    </string>

    <string name="BOOKMARK_CACHE_PASSPHRASE">ciDzDxGOxemVgEGcXpOa2oEufue/VH9A0FNjiXej3EZ2UP2+KToY7Zar6umEOTMrlNaVyFiDqDH0&#10;Pe+LREub+1uvDExWMC2jJ0ZjPojRE4sRomS0R/oBBtFv/sWJOjUKVF1A7dfBZu8OKScAMCO/h/jh&#10;+KzvpdS3/UnH7ynf9fL18cVxQm+fESwrZv4NN73Vo6s3gRhI2aHAyX58CFxh+HM06jtMb6EG1qbG&#10;Eu16APaOM4QZC9GHVqykTEMYLlAssD3FUppC3phbNHd3Ub6V1pty/slWiugnqdIuaCpsMEHiYxjT&#10;aEnMTCkKd+S87EYvghy5TZKa5+DtJOX6PSvYLg==&#10;    </string>

    <string name="SEARCH_HISTORY_CACHE_PASSPHRASE">Jz49gP8yRjoH2W9noVzXN2IcqyUI+nP0EKzWCWriErMZrea9BB/1uoaNrGvhsWbOrSvUT+hbMBlV&#10;LsgikkeKvnc88pvxRFxWktc833F0JitffPsmqu1JiNCmz0z50wut0p2fjjYs1bEkIncRod5AdegB&#10;z5+/yaWzVgFzAQ0cPlN4j/Bq4WfaUU627XrFhr+f3iVhiB3jyeN13XiDzz7daoXOlCGrFsW0bhql&#10;eSgkbIyoV9FmHnobPav3TICIkbOMKUHF1p2fuQqV2rNoYpeqw/t202/Ggl+pKhFxbCmHEDtEda5B&#10;MaV0lYDsthXTLxdySr0bk1oMBYOBqVX3WXcAzg==&#10;    </string>

s2가 존재하는 경우, arr_b는 s2가 base64로 디코딩된 문자열 배열이다.

위의 코드 중 arr_b1에 z.a(arr_b)를 넣는 부분에서 z.a는 아래와 같다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-56-41-image.png)

base64로 디코딩된 arr_b를 keystore에서 받아온 값을 이용해서 복호화한다.

암호 모드는 RSA/ECB/PKCS1Padding이다.

keystore에서 받아오는 개인키는 /data/misc/keystore/user_0/ 경로에 저장된다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-59-06-image.png)

 해당 경로에서 네이버 지도의 개인키와 .masterkey를 추출했다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-21-59-50-image.png)

마스터키의 버전이 3버전으로, 암호화된 구조체의 2버전에 대한 설명밖에 찾지 못했고, 구조체의 길이가 2버전과 달라 마스터키를 통해 개인키 구조체를 복호화하는 방법은 실패했다.

혹시나싶어 마스터키 버전 1의 복호화스크립트인 [GitHub - nelenkov/keystore-decryptor](https://github.com/nelenkov/keystore-decryptor) 를 사용하여 복호화를 시도하였으나 실패했다. 



##### 동적분석

최신 버전의 안드로이드에서는 TEE영역에서 키관리가 이루어져 램에서 키를 찾을 수 없다는 설명을 보았으나, 녹스의 안드로이드 버전이 7과 9버전을 사용하였고, 어플리케이션에서 데이터베이스를 사용하는 이상 db나 키에 대한 정보가 남아있을 것으로 추정되어 fridump3을 통해 메모리를 확인했다. 또한 sqlcipher를 안드로이드에서 구현 시 프로바이더로 keystore를 지원하지 않아, 키스토어에서 받아온 값을 저장한 뒤 그 값을 인자로 넣어 암호화를 구현한다고 나와있다.

참고: https://cellebrite.com/en/decrypting-databases-using-ram-dump-health-data/

디지털 포렌식 솔루션 제공사인 cellebrite에서 메모리포렌식을 이용하여 sqlcipher로 암호화된 데이터베이스를 복호화하는 것을 확인했다. 해당 블로그에서는 키와 IV(기기넘버)를 이용하여 AES-128로 암호화된 삼성 헬스 어플리케이션의 디비를 복호화하는 내용이 담겨있다. 그러나 갑작스러운 전개로 메모리의 스트링 중 96바이트의 문자열을 찾아야 된다고 나와 이유는 모르지만 똑같이 fridump3으로 문자열을 찾아봤으나 해당 96바이트의 문자열들은 암호화된 디비의 키가 아니었다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-22-23-43-image.png)

nmap의 개인키 헤더에 맞는 구조체가 메모리에 남아있는지 확인하였다.

![](C:\Users\dlehg\AppData\Roaming\marktext\images\2022-12-26-22-25-44-image.png)

구조체 형식에 맞는 문자열들이 몇군데 보였지만 찾은 키를 통해 디비를 복호화할 수 없었다. 또 ssl에 사용되는 것으로 추정된다.



## Results

네이버지도 어플리케이션의 3가지 데이터베이스 bookmark.db, route-history.db, search-history.db가 암호화된 것을 확인하여 복호화를 시도했다.

정적 분석을 통해 데이터베이스를 열기위해 진행되는 과정을 살펴보았고, keystore에 저장된 개인키를 통해 preferences에 저장된 passphrase를 복호화하여 데이터베이스를 복호화하는 것을 확인했다. 개인키를 복호화하기위해 개인키를 복호화할 수 있는 마스터키를 복호화하려고 했으나 실패하였다.

메모리덤프를 통해 메모리에 남아있는 키 구조체 또는 데이터베이스 복호화키를 구하려고 했으나 키 구조체는 남아있지 않았고, 암호화 키의 길이를 알 수 없어 키를 찾는 것도 실패했다.



## Discussions & Limitations

JEB를 이용한 안드로이드 어플리케이션 분석, JAVA로 짜여진 코드 분석, 암호화에 대한 부분을 처음 다뤄 기초부터 검색하느라 시간이 많이 소요되었고, 오히려 관련된 전문적인 지식을 습득할 시간이 부족했던 것 같다. 또한 마스터키-개인키 복호화과정에서 마스터키의 버전이 모두 과거버전으로 설명되어 있어 최신 버전의 마스터키에 대한 자료조사가 부족했거나, 아직 관련 연구가 이루어지지 않은 것으로 보인다.

또 복호화툴인 keystore-decryptor를 사용했으나 결과값이 제대로 나오지 않았다. 그래들로 설치하는 과정에서 shadow1.3 버전을 더이상 지원하지 않아 해결하기 위해 코드를 조금씩 수정했으나 그부분에서 문제가 생겼을 가능성도 있는 것 같다.

메모리 덤프에서는 셀레브레이트 블로그에서 본 96바이트를 찾는 이유에 대해 알 수 있었으면 암호화키를 찾는데 더 수월했을 것 같다.



## Conclusions and Future Directions

안드로이드 키스토어에서 받아오는 마스터키-개인키 구조체는 루트권한이 있는 사용자가 얻을 수 있었고, 마스터키 안의 (salt와 잠금 패스워드 값의 해시값)과 IV를 통해 마스터키 구조체를 복호화 할 수 있다. 마스터키 구조체 안의 마스터키를 통해 개인키 구조체를 복호화할 수 있으며, 개인키로 preference의 passphrase를 복호화하여 데이터베이스의 복호화키로 사용할 수 있다.

마스터키 구조체 버전3에 대한 조사를 진행하여 루트권한이 있는 사용자가 키스토어 키를 사용할 수 있는지에 대한 연구와 추가적인 메모리 분석을 통해 키를 얻을 수 있는지에 대한 연구를 진행할 예정이다.

안드로이드 버전 별 키스토어의 동작과정이 달라지므로 버전, 기종(제조사의 TEE영역이 달라지므로) 등의 여러가지 기계로 실험을 진행하면 좋을 것 같다.






