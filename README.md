# 특정 깃헙(Angular-cli) 이슈 목록과 상세 내용을 확인하는 웹 사이트 구축

1. [팀 소개 👫](#1-팀-소개-)
2. [프로젝트 소개 🚀](#2-프로젝트-소개-)
3. [기술 스택 🛠](#3-기술-스택-)
4. [구현 기능 📍](#4-구현-기능-)
5. [프로젝트 구조 🗂](#5-프로젝트-구조-)
6. [Best Practice 선정과정👩‍👦‍👦](#6-best-practice-선정과정)
7. [프로젝트 설치 및 실행 ✨](#7-프로젝트-설치-및-실행-)  

<br/>

- [🌍 배포 링크](https://2nd-assignment.vercel.app/)
- [📄 팀 노션](https://plain-airboat-3f4.notion.site/10-27-Todo-f9fb2a1265e54c33b0b73c306c230042)

<br />



## 1. 팀 소개 👫

- [이빛나 (팀장)](https://github.com/bitnaleeeee)
- [모상빈](https://github.com/Topbin2)
- [김진석](https://github.com/genuine-seok)
- [박우빈](https://github.com/Debonchocola)
- [이의연](https://github.com/strongpond)
- [조성호](https://github.com/CSH111)
- [전대원](https://github.com/eodnjs467)

<br />

## 2. 프로젝트 소개 🚀

- 개요 : 원티드 프론트엔드 프리온보딩 7기 1팀 과제 1-2 중 Best Practice
- 주제 : 특정 깃헙 레파지토리의 이슈 목록과 상세 내용을 확인하는 웹 사이트 구축
- 기간 : 2022.10.29 ~ 2022.10.30

<br />


## 3. 기술 스택 🛠

- React
- Typescript
- Styled-Components

<br />

## 4. 구현 기능 📍

- 이슈 목록 페이지 구현
  - 무한스크롤
  - 광고배너 삽입
- 상세 이슈 페이지 구현
- Context API를 활용한 이슈 상태관리 및 API 연동
- 데이터 요청 중 로딩 표시
- 에러 화면 구현
- 지정조건에 맞게 데이터 요청 및 표시
- 반응형 웹 구현

<br />

## 5. 프로젝트 구조 🗂

```bash
src
├── apis  // issue 관련 api service 요청
├── assets  // 전역 스타일링
├── components  // 공용 컴포넌트
├── context // issue 상태관리 context
├── hooks // scroll 관련 커스텀 훅
├── pages // 페이지 및 페이지 고유 컴포넌트
├── types // 공용타입 관리
└── utils // dateformatting, axios 관련 유틸 함수
```

<br/>


## 6. Best Practice 선정과정👩‍👦‍👦

  ### 선정이유 1 - 무한스크롤
  
  ```javascript
export const useInfiniteScroll = (
  onIntersect: () => void,
  options?: IntersectionObserverInit
) => {
  const [target, setTarget] = useState<Element | null>(null);

  const handleIntersect = useCallback(
    ([entry]: IntersectionObserverEntry[]) => {
      if (entry.isIntersecting) {
        onIntersect();
      }
    },
    [onIntersect]
  );

  useEffect(() => {
    const observer = new IntersectionObserver(handleIntersect, options);
    target && observer.observe(target);
    return () => {
      observer.disconnect();
    };
  }, [handleIntersect, target, options]);

  return [setTarget];
};
```
#### 코멘트
  - `Intersection Opserver API` 활용해 scrollEvent를 이용한 것에 비해 최적화를 효율적으로 적용한 점이 좋았습니다. - 박우빈
  - 재사용성을 위해 커스텀훅으로 분리한 부분이 좋았습니다. - 김진석

<br>

### 선정이유 2 - Context API를 활용한 API 연동
- `Context API` `useReducer` 를 활용하여 `Flux 패턴`으로 서버 데이터 관리 

```javascript
export enum IssueActionTypes {
  GET_ISSUE_LIST_SUCCESS = "GET_ISSUES_LIST_SUCCESS",
  GET_ISSUE_LIST_LOADING = "GET_ISSUES_LIST_LOADING",
  GET_ISSUE_LIST_ERROR = "GET_ISSUES_LIST_ERROR",

  GET_ISSUE_DETAIL_SUCCESS = "GET_ISSUES_DETAIL_SUCCESS",
  GET_ISSUE_DETAIL_LOADING = "GET_ISSUES_DETAIL_LOADING",
  GET_ISSUE_DETAIL_ERROR = "GET_ISSUES_DETAIL_ERROR",
}

const initialState: IssueCtxInitialState = {
  isLoading: false,
  isError: false,
  issueList: [],
  issueDetail: null,
};

const issueReducer = (
  state: IssueCtxInitialState,
  action: { type: IssueActionTypes; data?: Issue[] | Issue }
) => {
  switch (action.type) {
    case IssueActionTypes.GET_ISSUE_LIST_LOADING:
      return {
        ...state,
        isLoading: true,
        isError: false,
      };
    case IssueActionTypes.GET_ISSUE_LIST_SUCCESS:
      return {
        ...state,
        issueList: [...state.issueList, ...(action.data as Issue[])],
        isLoading: false,
        isError: false,
      };
    case IssueActionTypes.GET_ISSUE_LIST_ERROR:
      return {
        ...state,
        isLoading: false,
        isError: true,
      };
    case IssueActionTypes.GET_ISSUE_DETAIL_LOADING:
      return {
        ...state,
        isLoading: true,
        isError: false,
      };
    case IssueActionTypes.GET_ISSUE_DETAIL_SUCCESS:
      return {
        ...state,
        issueDetail: action.data as Issue,
        isLoading: false,
        isError: false,
      };
    case IssueActionTypes.GET_ISSUE_DETAIL_ERROR:
      return {
        ...state,
        isLoading: false,
        isError: true,
      };
    default:
      throw new Error("action type을 확인해주세요.");
  }
};

const IssueStateContext = createContext<IssueCtxInitialState>(initialState);
const IssueDispatchContext = createContext<React.Dispatch<{
  type: IssueActionTypes;
  data?: Issue[] | Issue;
}> | null>(null);

export const IssueProvider = ({ children }: { children: JSX.Element }) => {
  const [state, dispatch] = useReducer(issueReducer, initialState);
  return (
    <IssueStateContext.Provider value={state}>
      <IssueDispatchContext.Provider value={dispatch}>
        {children}
      </IssueDispatchContext.Provider>
    </IssueStateContext.Provider>
  );
};
```
#### 코멘트
  - 단방향 데이터 흐름으로 복잡성을 줄인점이 인상적이었습니다. - 조성호

<br>

## 7. 프로젝트 설치 및 실행 ✨

<br/>

1. Git Clone

```plaintext
$ git clone https://github.com/pre-onboading-2team/Week1_2_Issue_List.git
```

2. 프로젝트 패키지 설치

```plaintext
$ npm install
```

3. 프로젝트 실행

```plaintext
$ npm start
```

