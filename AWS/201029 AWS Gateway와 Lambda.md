## 201029 AWS Gateway와 Lambda

`201105 수정`

### Serverless != NonServer

- [서버리스](https://aws.amazon.com/ko/serverless/)란, 서버리스는 운영상의 책임을 클라우드 서비스(ex) AWS)로 전환하여 민첩성과 혁신을 높일 수 있도록 하는 클라우드의 네이티브 아키텍처이다. 

- 서버리스라고 해서 서버가 아예 없는 것은 아니다. 단지, 서버는 고려하지 않고 애플리케이션 구축 및 실행할 수 있는 아키텍처를 말한다.
- 사용하는 이유
  - 서버리스를 통해 증가된 민첩성과 낮은 총 소유 비용을 통해 최신 애플리케이션을 빌드할 수 있다.
  - 핵심 제품에 집중할 수 있다는 뜻.



## API Gateway

> 어떤 규모에서든 개발자가 API를 손쉽게 생성, 게시, 유지 관리, 모니터링 및 보안 유지할 수 있도록 하는 완전관리형 서비스입니다. API는 애플리케이션이 백엔드 서비스의 데이터, 비즈니스 로직 또는 기능에 액세스할 수 있는 "정문" 역할을 합니다.
>
> (API 게이트웨이를 먼저 생성할때는 추후 연결될 람다와 메서드 이름을 동일하게 만들어야 한다.)

- API Gateway 생성
  
  - RESTful API
- **리소스**를 만들고
- url 호출됐을 때 매핑될 **메서드**를 만들어야 한다.
  - 그 함수를 Lambda와 매핑!
  - [매핑 템플릿](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)
    
    - Content-Type
      
      - application/json
      
        ```js
        $input.json('$')
        ```
- 리소스를 다시 클릭하고 **CORS 활성화**
- 메서드를 클릭하고 API 배포
  - 새 스테이지
  - 스테이지 이름은 중요하지 않음
- URL 호출이 뜸



만약, 와일드카드를 주려면

- 리소스를 새로 만들면 됨 

- 계층이 여러개라면 각각 두 개

  - /welcome
  - {proxy+}

- 그리고 람다 코드를 수정하고 Deploy후 테스트해보기

  - ```js
    exports.handler = async (event) => {
        
        return {
                statusCode: 200,
                body: JSON.stringify(event.path)
        }
        
        return response;
    };
    ```





## Lambda

> 사실상 모든 유형의 애플리케이션이나 백엔드 서비스에 대한 코드를 별도의 관리 없이 실행할 수 있다.

- Lambda 생성

  - 새로운 권한 생성
  - 필요하다면 DynamoDB권한 추가 설정 가능

- 트리거 추가 

  >  (Lambda 생성 후 API Gateway에서 메소드와 연결했다면 별도로 추가하지 않아도 됨)

  - API 를 생성.
  
  - API 게이트웨이로 트리거 구성
  
- API ausg-serverless
  
- Lambda 함수에 API를 추가하여 함수를 호출하는 HTTP 엔드포인트를 생성합니다. API Gateway는 HTTP API 및 REST API와 같은 두 가지 유형의 RESTful API를 지원합니다. [자세히 알아보기](https://docs.aws.amazon.com/console/lambda/apigateway)
  
  - 코드 추가 후 디플로이. 접속하면 잘 됨
  
    - ```js
      exports.handler = async (event) => {
          
          if(event.httpMethod == 'GET'){
              return {
                  statusCode: 200,
                  body: JSON.stringify("get")
              }
          } else {
              return {
                  statusCode: 200,
                  body: JSON.stringify("post")
              }
          }
          
          return response;
      };
      ```



- [Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)
  - Lambda 함수를 구성하여 코드 및 콘텐츠를 레이어 형태로 가져올 수 있다.
  - 라이브러리, 사용자 지정 런타임 또는 기타 종속정이 포함된 ZIP 아카이브이다.
  - 레이어를 사용하면 배포 패키지에 포함할 필요없이 함수에서 라이브러리를 사용할 수 있다.



- [Cold Start](https://alphahackerhan.tistory.com/29)

  - Lambda의 근간이 되는 기술인 컨테이너라는 개념은 스케일링하는데 있어 혁신적이라고 여겨진다. Scale-out하는 상황에서, AWS EC2와 같이 가상 OS가 새로 생성되는 방식이 아닌, OS 위에 돌아가는 독립적인 하나의 프로그램을 생성하는 개념이기 때문이다.

  - 때문에 당연히 훨씬 가볍고, 낮은 사양으로 셋팅하더라도, 부하가 생기는 순간에 컨테이너가 빨리 뜨면서 Scale-out되고, 부하를 다 받아낼 수 있을 거라고 기대하게 되지만, 그렇지 않은 경우가 많이 존재한다.

  - **그렇게 되지 않는 경우 = 빠르게 Container가 생성되지 않는 경우 = Cold Start**

  - 결국 Cold start는 완전히 식어버린 컴퓨터를 새로 킨다는 의미이다.

    - 네트워크쪽에서 다시 연결하느라 시간이 오래 걸리게 된다는 것.

  - 해결방법

    1. Lambda 사양을 높인다.
       - 메모리 양을 증가 = 비용 증가

    2. Provisioned Concurrency를 사용한다.
       - Warm start를 할 수 있게 도와주는 서비스
       - 비쌈
    3. 자체적인 Warm start
       - CloudWatch Event의 스케쥴러를 이용해 한번씩  5분마다 한번씩 요청을 주는 방법이 있다.
       - 배포 환경이 따로 구성되어있다면 그 곳에 스케쥴러 적용
       - 비교적 비용이 덜 듦. 귀찮을 수 있음.
    4. 이상적으로 일정량의 요청이 꾸준히 들어오는 것






## 추가적인 내용

### IAM 권한 

- 사용자(User)
  - 직접 권한을 행사할 때 사용
  - Access secret Credential이 필요함
  - 장점
    - AWS 내부가 아닌 외부에서도 사용할 수 있다.
- 역할(Role)
  - 서비스에 붙일 때 사용
  - Credential이 필요하지 않음
  - 장점
    - 자격증명 관리할 필요가 없다.
    - ex) CloudWatch(모니터링 서비스)를 사용하고 싶을 때 람다 안에 유저 크레덴셜을 넣는게 아니라 어떤 롤을 사용할지 접근 권한이 있는 롤을 설정만 해주면 람다가 권한을 임시로 얻어옴.



### DynamoDB

> NoSQL 데이터베이스

DynamoDB까지 연결하는 예제는 인터넷 참고. 👉 [링크](https://velog.io/@nari120/DynamoDB-Lambda-%EC%98%88%EC%A0%9C)

**Table**

| 속성  | 자료형  |
| ----- | ------- |
| id    | integer |
| name  | string  |
| email | string  |



**Get** [자습서](https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/dynamodb-example-document-client.html)

- 데이터베이스 selectAll 메소드를 담당하는 dbRead()의 개념은 [Stack overflow](https://stackoverflow.com/questions/44589967/how-to-fetch-scan-all-items-from-aws-dynamodb-using-node-js) 참고

```js
var AWS = require('aws-sdk')

AWS.config.update({
    region: 'ap-northeast-2',
    endpoint: "http://dynamodb.ap-northeast-2.amazonaws.com"
})
const docClient = new AWS.DynamoDB.DocumentClient();

async function dbRead(params) {
    let promise = docClient.scan(params).promise();
    let result = await promise;
    let data = result.Items;
    if (result.LastEvaluatedKey) {
        params.ExclusiveStartKey = result.LastEvaluatedKey;
        data = data.concat(await dbRead(params));
    }
    return data;
}

exports.handler = async function(event, context, callback) {
    console.log(event);
    var params = {
        TableName: "ausg"
    };
    
    let data = await dbRead(params);
    return {
        body : data
    }
}
```



**Post** 

```js
var AWS = require('aws-sdk')

AWS.config.update({
    region: 'ap-northeast-2',
    endpoint: "http://dynamodb.ap-northeast-2.amazonaws.com"
})

const docClient = new AWS.DynamoDB.DocumentClient();
exports.handler = function(event, context, callback) {
    console.log(event);
    var params = {
        TableName: "ausg",
        Item: {
            "id": event.id,
            "name": event.name,
            "email": event.email
        }
    };

    docClient.put(params, function(err, data) {
        if(err){
            callback(err, null);
        } else{
            callback(null, data);
        }
    })
}
```

- API Gateway 매핑 템플릿

  ```json
  $input.json('$')
  ```

  