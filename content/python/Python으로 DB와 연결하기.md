---
tags:
  - python
date: 2024-03-25
---
# sqlite로 DB 생성

간단하게 DB와 Table을 만듭니다.
![[Pasted image 20240325152340.png|700]]

# Python을 이용해 DB에 값 삽입하기

![[Pasted image 20240325160252.png]]
파이썬에서 DB를 연결하여 데이터를 입력하는 과정은 위와 같습니다.
sqlite3 모듈을 import 하여, sqlite를 연결한후, 사용자로부터 값을 입력받아 SQL문으로 DB에 값을 입력 저장합니다.

```
import sqlite3

  

con, cur = None, None
con = sqlite3.connect("C:/Users/COM/Desktop/PythonWorkspace/sqlite/googledb")
cur = con.cursor()

  

while (True) :
    data1 = input("사용자ID ==> ")
    if data1 == "":
        break
    data2 = input("사용자이름 ==> ")
    data3 = input("이메일 ==> ")
    data4 = input("출생연도 ==> ")
    sql = "INSERT INTO userTable VALUES('"+ data1 +"','"+ data2 + "', '" + data3 + "', " + data4 +")"
    cur.execute(sql)
  
con.commit()
con.close()
```

위 코드에서 sql 구문을 작성을 신경써줘야 합니다. 
아래와 같은 sql 구문을 만들어주기 위해서 python 코드처럼 작성해 주었습니다.
```
INSERT INTO userTable VALUES('zito','zito','zito@naver.com',20)
```


파이썬 코드를 실행시켰습니다.
콘솔을 이용해 Table에 삽입할 값을 입력해 주었습니다
![[Pasted image 20240325160452.png]]

DB에서 확인해보면, 잘 들어있습니다.
![[Pasted image 20240325160534.png]]



# Python을 이용해 DB의 값 조회하기

![[Pasted image 20240325161901.png|700]]

Python을 이용해 DB의 값을 조회하는 과정은 위과 같습니다.
앞서 작성한 Python을 이용해 DB에 값을 작성하는 것과 유사합니다.

Sqlite3 Module을 사용하여 연결 한후, Select문으로 커서에 값을 가져옵니다.
가져온 값에서 fetchone() 함수를 이용해 한 행씩 가져와서 출력합니다.

아래 코드를 보면서 좀 더 자세히 살펴보겠습니다.


```
import sqlite3


con, cur = None, None
con = sqlite3.connect("C:/Users/COM/Desktop/PythonWorkspace/sqlite/googledb")
cur = con.cursor()
  
cur.execute("SELECT * FROM userTable") # 커서에 Select로 조회한 결과 한꺼번에 저장
print("사용자ID     사용자이름      이메일      출생연도")
print("-----------------------------------------------")
  
while (True) :
    row = cur.fetchone() # 커서에서 한 행씩 접근
    if column == None :
        break
    data1 = row[0]
    data2 = row[1]
    data3 = row[2]
    data4 = row[3]
    print("%5s  %15s  %15s  %d" %(data1, data2, data3, data4))

con.close()
```

위 코드를 통해 DB에 접근해 값을 조회하여 커서에 저장합니다.
저장 후에 한 행씩 접근하여 출력합니다.

코드를 실행하면 잘 출력됩니다.
![[Pasted image 20240325162527.png]]

# Python을 이용해 DB에서 Delete하기

Insert, Select를 해보았으니, 이제 Delete를 Python을 이용해 해보겠습니다.
동작 방법은 Insert와 유사합니다.
SQL만 Delete에 맞게 작성해 주었습니다.

삭제하기 전 테이블입니다.
![[Pasted image 20240325170615.png]]

```
import sqlite3

  
con, cur = None, None
con = sqlite3.connect("C:/Users/COM/Desktop/PythonWorkspace/sqlite/googledb")
cur = con.cursor()

data1 = input("삭제할ID ==> ")
sql = "DELETE FROM userTable WHERE id = '" + data1 + "'"
cur.execute(sql)
con.commit()
print("id = %s의 row가 삭제되었습니다." %data1)
con.close()
```

위 코드를 동작시켜 id=john1인 행을 삭제했습니다.
![[Pasted image 20240325170711.png]]

확인해보면 john1이 삭제되었음을 알 수 있습니다.
![[Pasted image 20240325170645.png]]

### 삭제후 table을 바로 확인하기

위 실습에서는 삭제후, 직접 DB로 접근해서 table을 확인했습니다.
삭제후 삭제가 된 테이블을 바로 볼 수 있게 코드를 추가해 보겠습니다.

```
import sqlite3

  

con, cur = None, None
con = sqlite3.connect("C:/Users/COM/Desktop/PythonWorkspace/sqlite/googledb")
cur = con.cursor()
 

data1 = input("삭제할ID ==> ")
sql = "DELETE FROM userTable WHERE id = '" + data1 + "'"
cur.execute(sql)
con.commit()
print("id = %s의 row가 삭제되었습니다." %data1)

cur.execute("select * from userTable")
print("사용자ID     사용자이름      이메일      출생연도")
print("-----------------------------------------------")
while (True) :
    row = cur.fetchone()
    if row == None :
        break
    data1 = row[0]
    data2 = row[1]
    data3 = row[2]
    data4 = row[3]
    print("%5s  %15s  %15s  %d" %(data1, data2, data3, data4))

con.close()
```

삭제후, 삭제이후의 테이블이 바로 보입니다.
![[Pasted image 20240325171350.png]]


# Python을 이용해 DB Update 하기

Insert, Select, Delete를 해 보았습니다. 
마지막으로 Update를 해보겠습니다.
흐름은 Delete와 유사합니다.

ID를 입력받고, 입력받은 ID의 출생연도를 바꿔 보겠습니다.

```
import sqlite3

con, cur = None, None
con = sqlite3.connect("C:/Users/COM/Desktop/PythonWorkspace/sqlite/googledb")
cur = con.cursor()

data1 = input("출생연도를 바꾸고 싶은 ID ==> ")
data2 = input("바꿀 출생연도 ==> ")
sql = "Update userTable set birthYear = " + data2 + " WHERE id = '" + data1 + "'"
cur.execute(sql)
con.commit()
print("id = %s의 row가 업데이트되었습니다.." %data1)

cur.execute("select * from userTable")

print("사용자ID     사용자이름      이메일      출생연도")
print("-----------------------------------------------")
while (True) :
    row = cur.fetchone()
    if row == None :
        break
    data1 = row[0]
    data2 = row[1]
    data3 = row[2]
    data4 = row[3]
    
    print("%5s  %15s  %15s  %d" %(data1, data2, data3, data4))

con.close()

```

바꾸기 전 테이블
![[Pasted image 20240325172353.png]]

바뀐 후의 테이블
![[Pasted image 20240325172418.png]]

john3의 출생연도가 2003으로 바뀌며 회춘(?)했습니다.

