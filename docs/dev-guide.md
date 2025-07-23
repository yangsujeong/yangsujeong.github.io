---
id: dev-guide
title: 개발 가이드
---

# 개발 가이드

## 전체 흐름

```
[1] 화면 요청
[2] Frontend - sample-view.vue
[3] Frontend - API 호출 (sample-view.ts)
[4] Backend - Controller (SampleMenuViewController)
[5] Backend - Service (SampleMenuViewService)
[6] Backend - Mapper Interface (SampleMenuViewMapper)
[7] Backend - Mapper XML (SampleMenuViewMapper.xml)
[8] DB 쿼리 실행,결과 반환
[9] Backend - 결과 응답
[10] Frontend - 결과 화면 출력
```

## 디렉토리 및 주요 파일 구조

### Backend

```jsx
back
└ root
   └ bo
      └ src
         └ main
            ├ java
            │  └ sample.ustraframework.java.bo.sampleMenu.view
            │     ├ SampleMenuViewController.java   
            │     ├ SampleMenuViewService.java      
            │     ├ SampleMenuViewMapper.java       
            │     └ SampleMenuViewModel.java       
            └ resources
               └ mapper
                  └ SampleMenuViewMapper.xml       
```

| 파일 | 설명 |
| --- | --- |
| `SampleMenuViewController.java` | POST 요청을 받아 Criteria 매핑 후 Service 호출 |
| `SampleMenuViewService.java` | Mapper 호출, 페이징 정보 처리 |
| `SampleMenuViewMapper.java` | MyBatis 매핑 인터페이스 |
| `SampleMenuViewMapper.xml` | 실제 SQL 처리 및 조건 분기 |
| `SampleMenuViewModel.java` | Criteria 및 SearchValue 중첩 구조 정의 |
- `SampleMenuViewController.java`

  : 클라이언트에서 전달한 JSON이 Criteria 객체로 매핑됨


```java
@PostMapping("/list")
public PaginationList<SampleMenuViewModel> getCodeList(@RequestBody SampleMenuViewModel.Criteria criteria) {
    return sampleMenuViewService.getList(criteria);
}
```

- `SampleMenuViewService.java`

  : 페이징 요청 정보와 검색 조건을 Mapper로 전달


```java
public PaginationList<SampleMenuViewModel> getList(SampleMenuViewModel.Criteria criteria) {
    return sampleMenuViewMapper.select(criteria.getPaginationRequest(), criteria);
}
```

- `SampleMenuViewMapper.java`

  : XML에서는 `#{criteria.searchValue.sampleId}`로 접근


```java
PaginationList<SampleMenuViewModel> select(PaginationRequest pagination, SampleMenuViewModel.Criteria criteria);
```

- `SampleMenuViewMapper.xml`

  : 검색 조건과 정렬 조건을 동적으로 적용


```xml
<select id="select" resultType="SampleMenuViewModel">
    SELECT sample_id, sample_name
    FROM ustra_sample_view
    WHERE 1 = 1
    <if test="criteria.searchValue.sampleId != null and criteria.searchValue.sampleId != ''">
        AND sample_id = #{criteria.searchValue.sampleId}
    </if>
    ORDER BY ${criteria.sortItem} ${criteria.sortDirection}
</select>
```

### Frontend

```jsx
front
└ bo
   ├ pages/sample-menu/sample-view.vue         
   ├ app/services/sample-view.ts               
   ├ types/sampleViewList.ts                   
   └ components/ustra/sample_menu/sample_view/
      └ index.vue                              
```

| 파일 | 설명 |
| --- | --- |
| `sample-view.vue` | 화면 렌더링, 사용자 이벤트 처리, API 호출 |
| `sample-view.ts` | API 호출 정의 (`$ustra.api.call`) |
| `sampleViewList.ts` | Criteria 및 하위 SearchValue 타입 정의 |
| `index.vue` | 테이블, 검색조건 등 화면 재사용 UI 컴포넌트 |
- `sample-view.vue`

: 사용자 입력값을 기준으로 `criteria` 객체를 구성

```jsx
const criteria: SampleViewCriteria = {
  searchValue: {
    sampleId: 'TEST001',
    sampleName: '테스트'
  },
  sortItem: 'sample_id',
  sortDirection: 'asc',
  paginationRequest: {
    currentPage: 1,
    pageSize: 20
  }
}
```

- `sample-view.ts`

: 위 criteria 객체를 POST 방식으로 API 호출

```jsx
await $ustra.api.call({
  url: '/api/sample-menu/sample-view/list',
  method: 'POST',
  data: criteria
})
```

## 화면 메뉴 추가

- 메뉴 관리 > 상위 메뉴 추가

![image.png](/img/guide/image.png)

- 메뉴 관리 > 하위 메뉴 추가

![image.png](/img/guide/image1.png)

![image.png](/img/guide/image2.png)

- 권한 관리 > 그룹별 권한 설정

![image.png](/img/guide/image3.png)

- 캐시 제거 후 재 로그인

![image.png](/img/guide/image4.png)

- 샘플 메뉴 조회

![image.png](/img/guide/image5.png)