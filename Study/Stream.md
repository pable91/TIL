# Stream api 설명

# 중간 연산
### map
- 요소들을 조건에 맞게 변환하는 메서드

### filter
- 스트림내에 있는 요소에 대해 필터링하는 메서드

### distinct
- 중복된 데이터가 존재하면 제거하는 메서드
- 사용자 정의 클래스를 Stream에 사용한다고하면 equals와 hashCode를 오버라이드해야만 distinct()가 제대로 적용된다.

### mapToInt(), mapToLong(), mapToDouble(), mapToObj()
- 일반 Stream 객체를 원시 Stream으로 바꾸거나 그 반대의 작업이 필요할때 사용하는 메서드


---

# 결과 연산

### collect(Collector collector)
- Stream 데이터를 원하는 자료형(List,Set,Map 등)으로 변화해주는 메서드
- 매개변수로는 Collector가 와야하는데, 사용자가 직접 구현하는 Collector 인터페이스와 이미 정의해놓은 Collectors 클래스가 매개변수로 갈 수 있다. 

1. Collectors.toList() : 리스트로 반환
2. Collectors.toSet() : set으로 반환
3. Collectors.joining() : 작업한 결과를 String 1개로 이어붙이기 할 수 있는 메서드
4. Collectors.groupingBy() : 작업한 결과를 특정 그룹으로 묶을때 사용하는 메서드, 결과는 Map으로 반환

### Match
- 요소들이 특정한 조건을 충족하는지 검사하는 메서드

1. anyMatch : 1개의 요소라도 해당 조건을 만족하는지 체크
2. allMatch : 모든 요소가 해당 조건을 만족하는지 체크
3. nonMatch : 모든 요소가 해당 조건을 만족하지 않는지 체크

### forEach
- 요소들을 대상으로 특정한 연산을 수행하고 싶을때 사용하는 메서드 
- peek()는 중간연산이고 Stream을 반환하는 반면 forEach는 최종연산으로 반환값이 존재하지않는다.



