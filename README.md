## 특정 깃헙(Angular-cli) 이슈 목록과 상세 내용을 확인하는 웹 사이트 구축

## 배포 링크 : [바로가기 클릭](https://2nd-assignment.vercel.app/)


## 실행결과

<img src="https://user-images.githubusercontent.com/99943583/206370641-506812bb-70eb-4d4e-bf70-0d371a85402a.gif">

#### STACK
<img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=React&logoColor=white"> <img src="https://img.shields.io/badge/TypeScript-3178C6.svg?&style=for-the-badge&logo==TypeScript&logoColor=white" ><img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=JavaScript&logoColor=white">
<img src="https://img.shields.io/badge/styled components-DB7093?style=for-the-badge&logo=styled-components&logoColor=white">
<img src="https://img.shields.io/badge/React_Router-CA4245?style=for-the-badge&logo=React Router&logoColor=white">

## 팀 소개

- [이빛나 (팀장)](https://github.com/bitnaleeeee)
- [모상빈](https://github.com/Topbin2)
- [김진석](https://github.com/genuine-seok)
- [박우빈](https://github.com/Debonchocola)
- [이의연](https://github.com/strongpond)
- [조성호](https://github.com/CSH111)
- [전대원](https://github.com/eodnjs467)

<br>

## 프로젝트 소개 

- 목표 : 특정 깃헙 레파지토리의 이슈 목록과 상세 내용을 확인하는 웹 사이트 구축
- 기간 : 2022.10.29 ~ 2022.10.30 
- 주관 : 원티드 

<br>

## 구현 사항

- [x] 이슈 목록 페이지 구현 : 무한스크롤, 광고배너 삽입
- [x] 상세 이슈 페이지 구현
- [x] Context API를 활용한 이슈 상태관리 및 API 연동
- [x] 데이터 요청 중 로딩 표시
- [x] 에러 화면 구현
- [x] 지정조건에 맞게 데이터 요청 및 표시
- [x] 반응형 웹 구현


<br />

## 폴더 구조 

```
📦 src
├── 📂  apis
├── 📂  assets
├── 📂  components
├── 📂  context
├── 📂  hooks
├── 📂  pages
├── 📂  types
├── 📂  utils

```

- apis : issue 관련 api service 요청하는 폴더입니다
- assets : 전역 스타일링 관리 폴더입니다
- components : 공용 컴포넌트 입니다
- context : issue 상태관리 context 폴더 입니다
- hooks : scroll 관련 커스텀 훅 폴더입니다
- pages : 페이지 관리 컴포넌트 입니다
- types : 공용타입 관리 폴더 입니다
- utils : dateformatting, axios 관련 유틸 함수 폴더입니다


<br/>


## 주요 기능

  ### 무한스크롤
  
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

<br>

### Context API를 활용한 API 연동
  #### `Context API` `useReducer` 를 활용하여 `Flux 패턴`으로 서버 데이터 관리 


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

<br>

## 프로젝트 설치 및 실행

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

