## 200924 AWS-with-github-action

S3 + Cloud Front

Cloud Front 로 페이지 만들고, Route53 으로 도메인 연결한뒤 S3버킷 퍼블릭으로 생성하고 연결하기!

이번 실습에서는 Route53 생략하고 CloudFront만 생성해도 도메인 나와서 그거 이용함.



## S3 버킷 생성

- 퍼블릿 버킷
- index.html 로컬 생성 후 업로드
- index.html 파일 또한 퍼블릭으로 설정

```index.html
<head>
    <meta charset="UTF-8">
</head>
<body>
	<h1>hello, world</h1>
	<h2>안녕하세요!</h2>
</body>
```



## CloudFront

- **Origin Domain Name** 
  - 여기에 S3버킷의 이름을 복붙
    - 과거에는 아래 나오는 버킷을 넣으면 안된다고 하심. 이제는 될 수도??
- **Origin Path**
  - /
  - /가 기본이라서 다른데 클릭하면 없어짐ㅋㅋ



클라우드 프론트에는 도메인을 지정할 수 있음. 나는 기본으로 함.



## 캐시 무효화

html 한글이 깨져서 코드 수정 후 다시 업로드. -> 퍼블릭으로 변경

index.html 수정을 하고나면, 아무리 기다려도 안됨. -> Cloud Front의 캐시를 무효화해야됨.

![화면 캡처 2020-09-24 211808](https://user-images.githubusercontent.com/44438366/94156721-5d8bbd80-febb-11ea-9fb6-fd23cb0f218d.png)

Create Invalidation에서  /index.html하면 안되고 /*해야됨!!

