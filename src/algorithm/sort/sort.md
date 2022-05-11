## 정렬 알고리즘

- 특정 값을 기준으로 데이터를 순서대로 배치하는 방법
- 구현 난이도가 쉽고 속도가 느림
    - 버블 정렬, 삽입 정렬, 선택정렬
- 구현 난이도가 어렵고 속도가 빠름
  - 합병 정렬, 힙 정렬, 퀵 정렬, 트리 정렬
- 하이브리드 정렬
  - 팀 정렬, 블록 병합 정렬, 인트로 정렬
- 기타 정렬
  - 기수 정렬, 카운팅 정렬, 셸 정렬, 보고 정렬

---

| 구분 | 버블 정렬 | 삽입 정렬 | 선택 정렬 | 
| :---: | :---: | :---: | :---: |
| 설명 | 인접한 데이터를 비교하며 자리를 바꾸는 형식 | 앞의 데이터를 정렬 해가면서 삽입 위치를 찾아 정렬하는 방식 | 최소 또는 최대값을 찾아 가장 앞 또는 가장 뒤부터 정렬하는 방식
| 시간 복잡도 | O(n^2) | O(n^2) | O(n^2) |
| 보조 메모리 | 1 | 1 | 1 |
| 안정성  | O | O | X |

| 구분 |  합병 정렬 |  힙 정렬 | 퀵 정렬 | 트리 정렬 |
| :---: | :---: | :---: | :---: | :---: | 
| 설명 | 배열을 길이가 1이 될때까지 계속 분할한 뒤 인접한 부분끼리 정렬하면서 합병하는 방식 | 힙 자료구조 형태의 정렬, 기존 배열을 최대힙 으로 구조 변경 후 정렬 | 임의 기준 값을 정하고 그 값을 기준으로 좌우로 분할해 정렬 | 이진 탐색 트리(BST)를 만들어 정렬 |
| 복잡도 | O(nlogn) | O(nlogn) | O(n^2)| O(nlogn) |
| 보조 메모리 | n | 1 | log n | n |
| 안정성  | O | X | X | X |

| 구분 |  기수 정렬 | 계수 정렬 | 셸 정렬 | 
| :---: | :---: | :---: | :---: |  
| 설명 | 낮은 자리 수부터 정렬, 각 원소 간의 비교를 하지 않아 빠르지만 기수 테이블을 위한 메모리 필요 | 숫자끼리 비교하지 않고 카운트를 세 정렬하는 방식, 카운팅을 위한 메모리 필요 | 삽입 정렬 보완, 이전의 모든 데이터와 비교하지 않고 일정 간격을 두고 비교 | 
| 복잡도 | O(dn) | O(n+k) | O(n^2) |
| 보조 메모리 | n + k | n + k | 1 |
| 안정성  | O | O | X |

`d : 최대 자릿수`
`k : 정렬 대상 데이터 중 최대 값`

`삽입 정렬의 약점 : 구성된 데이터에 대해서 앞의 데이터와 하나씩 비교하며 모두 교환이 필요`

---

### 안정성
- 중복된 값이 있을 경우 초기 입력된 순서가 정렬 이후에도 유지되는 것을 말함(보통 key, value 이루어진 데이터)
- 초기 입력된 순서가 정렬 이후에 유지되지 않는다면 해당 정렬 알고리즘은 불안정 한 것

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&amp;fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkSMMG%2FbtqXwdOIhlD%2FXuLlErsdCRkCuAp2ebPqG1%2Fimg.jpg" width="80%" alt="https://www.geeksforgeeks.org/stability-in-sorting-algorithms/"/>