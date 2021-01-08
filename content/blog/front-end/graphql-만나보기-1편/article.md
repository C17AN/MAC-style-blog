---
title: "[프론트엔드] - GraphQL 사용하기 : 1편"
date: 2020-12-10 12:21:13
category: "프론트엔드"
thumbnail: "../graphql.png"
excerpt: "REST api를 보완하는 새로운 데이터 페칭방식 사용하기"
color: "#1EC800"
draft: false
---

![thumbnail](../graphql.png)

서버로부터 데이터를 받아와 빠르고 손실 없이 클라이언트에 전송하는 작업은 어떤 서비스를 만들 때든 필수불가결합니다.  
대표적인 API 제공 패러다임인 RESTful api[^1]는 오랜 시간동안 사랑받으면서 자리를 굳건히 다지는데 성공했지만, 어느 기술이 그렇듯 완벽할 수는 없었습니다.  
오늘은 REST API가 가지고 있던 문제점과 함께 GraphQL이 이러한 문제를 어떻게 해결할 수 있는지 소개해보는 시간을 갖겠습니다.

## 1. 데이터가 너무 많아요! - 오버페칭

> 1. 샘플 RESTful api 서버로는 [스타워즈 RESTful api 서버 (https://swapi.dev/)](https://swapi.dev/) 를 사용합니다.
> 2. 샘플 GraphQL api 서버로는 [스타워즈 GraphQL api 서버(https://graphql.org/swapi-graphql)](https://graphql.org/swapi-graphql) 를 사용합니다.

한번 기존의 RESTful한 엔드포인트를 활용해 스타워즈 등장인물의 데이터를 불러와 보겠습니다.  
아래는 https://swapi.dev/api/people/1 로 보낸 GET 요청의 응답 json 파일입니다.

```json
{
  "name": "Luke Skywalker",
  "height": "172",
  "mass": "77",
  "hair_color": "blond",
  "skin_color": "fair",
  "eye_color": "blue",
  "birth_year": "19BBY",
  "gender": "male",
  "homeworld": "http://swapi.dev/api/planets/1/",
  "films": [
    "http://swapi.dev/api/films/1/",
    "http://swapi.dev/api/films/2/",
    "http://swapi.dev/api/films/3/",
    "http://swapi.dev/api/films/6/"
  ],
  "species": [],
  "vehicles": [
    "http://swapi.dev/api/vehicles/14/",
    "http://swapi.dev/api/vehicles/30/"
  ],
  "starships": [
    "http://swapi.dev/api/starships/12/",
    "http://swapi.dev/api/starships/22/"
  ],
  "created": "2014-12-09T13:50:51.644000Z",
  "edited": "2014-12-20T21:17:56.891000Z",
  "url": "http://swapi.dev/api/people/1/"
}
```

어떤가요? 여러 정보가 상세히 담겨 있네요.  
하지만 만약 클라이언트에서 필요한 정보가 주인공의 이름과 성별뿐이라면 이 데이터는 너무 과하지 않을까요?  
클라이언트가 필요로 하는 데이터는 단 둘 뿐인데, 엔드포인트에서 받아본 데이터 항목은 자그마치 16개에 달하죠.  
바로 이렇게 불필요한 정보들을 너무 많이 받아오는 경우를 **_오버페칭_** 이라고 합니다.

그럼 GraphQL은 이 문제를 어떻게 해결할 수 있을까요?

```graphql
query {
  person(personID: 1) {
    name
    gender
  }
}
```

맨 위에 있는 샘플 GraphQL 서버에 들어가서 이렇게 요청을 보내봅시다.

```json
{
  "data": {
    "person": {
      "name": "Luke Skywalker",
      "gender": "male"
    }
  }
}
```

그럼 놀랍게도 요청한 이름과 성별 그 이상, 그 이하도 받지 않고 원하던 데이터만을 응답으로 받을 수 있습니다.  
불필요한 데이터를 가져오지 않아 응답 속도 역시 빨라질 여지가 있고, 오버페칭된 데이터를 따로 가공할 일도 없습니다.

## 2. 데이터가 모자라요! - 언더페칭

클라이언트를 만들던 중 주인공의 이름, 성별뿐만 아니라 탑승한 우주선들의 이름이 필요하다는 사실을 깨달았습니다.

```json
"starships": [
    "http://swapi.dev/api/starships/12/",
    "http://swapi.dev/api/starships/22/"
],
```

RESTful api라면 주인공의 정보를 불러온 다음 "starships" 키에 해당하는 배열에 담긴 각각의 엔드포인트로 요청을 추가로 보내야 합니다.

```json
{
  "name": "X-wing",
  "model": "T-65 X-wing",
  "manufacturer": "Incom Corporation",
  "cost_in_credits": "149999",
  "length": "12.5",
  "max_atmosphering_speed": "1050",
  "crew": "1",
  "passengers": "0",
  "cargo_capacity": "110",
  "consumables": "1 week",
  "hyperdrive_rating": "1.0",
  "MGLT": "100",
  "starship_class": "Starfighter",
  "pilots": [
    "http://swapi.dev/api/people/1/",
    "http://swapi.dev/api/people/9/",
    "http://swapi.dev/api/people/18/",
    "http://swapi.dev/api/people/19/"
  ],
  "films": [
    "http://swapi.dev/api/films/1/",
    "http://swapi.dev/api/films/2/",
    "http://swapi.dev/api/films/3/"
  ],
  "created": "2014-12-12T11:19:05.340000Z",
  "edited": "2014-12-20T21:23:49.886000Z",
  "url": "http://swapi.dev/api/starships/12/"
}
```

게다가 각각의 우주선 정보는 또다른 큰 객체에 담겨 있는데 우주선 정보 객체에서 필요한 데이터는 우주선의 이름뿐입니다.  
세상에, 그럼 주인공의 정보를 요청할 때 1번, 두 대의 우주선 정보를 불러올 때 2번, 총 3번의 요청을 보내야 하네요.  
거기에 딸려온 불필요한 데이터 가공은 덤으로 해결할 문제고요.

바로 이렇게 필요한 데이터를 한 번의 요청으로 불러오지 못하는 문제를 **_언더페칭_** 이라고 부르는데요, GraphQL을 활용하면 언더페칭 문제도 깔끔히 해결할 수 있습니다.

```graphql
query {
  person(personID: 1) {
    name
    gender
    starshipConnection {
      starships {
        name
      }
    }
  }
}
```

주인공의 이름, 성별, 탑승 우주선 이름을 묻는 쿼리문을 위와 같이 구성한 후 요청을 보내봅시다.

```json
{
  "data": {
    "person": {
      "name": "Luke Skywalker",
      "gender": "male",
      "starshipConnection": {
        "starships": [
          {
            "name": "X-wing"
          },
          {
            "name": "Imperial shuttle"
          }
        ]
      }
    }
  }
}
```

단 한 번의 요청으로 주인공의 정보와 함께 탑승한 우주선들의 이름이 담긴 객체를 받아온 모습입니다.

## 3. 엔드포인트 관리

REST api의 또다른 단점은 유연성이 부족하다는 점입니다.  
REST api 서버를 만들 때 주인공의 정보를 가공한 후 이름과 생일만을 제공하는 엔드포인트를 따로 만들 수도 있겠지만, 주인공의 이름 + 생일 + 피부색 / 이름 + 피부색 / 이름 + 생일 + 탑승 우주선 정보 ... 등 다양한 조합이 필요하다면 이렇게 수많은 조합에 일일히 대응하는 엔드포인트를 과연 효과적으로 관리할 수 있을까요?

```js
https://sampleserver.com/NameWithBirth/1 // 주인공 이름 & 생일 요청 주소
https://sampleserver.com/NameWithBirthAndSkin/1 // 주인공 이름 & 생일 & 피부색 요청 주소
https://sampleserver.com/NameWithBirthAndSpaceship/1 // 주인공 이름 & 생일 & 탑승 우주선 정보 요청 주소
...외에도 수많은 조합
```

새로운 엔드포인트와 api 문서를 계속해서 최신화할 수 있다면 그렇게 해도 되지만, 중요한 점은 GraphQL을 활용하면 더이상 그럴 필요가 없다는 것입니다!  
GraphQL을 사용하면 설계상 엔드포인트가 대부분 하나로 끝나게 되고 데이터 형식의 일관성도 지킬 수 있습니다.

GraphQL이 아직 REST 방식을 완전히 대체할 수는 없겠지만, 다음 프로젝트에서 api 서버를 만들어야 한다면 꼭 GraphQL을 사용해보고 싶습니다. 😁

[^1]: GET, POST, PUT, DELETE 등의 행동을 통해 각각의 라우트로 원하는 동작을 요청할 수 있는 방식
