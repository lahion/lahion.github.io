---
title: (Django) filter 올바르게 사용하기 2편
author: 쿠로
date: 2024-12-11
description: filter 올바르게 써보기.
---

1편에 이어 2편을 진행하겠습니다.

### 문제사항

~~1. DB에 저장된 데이터들을 가공하여 통계를 내는 과정에서 외래키로 연결된 필드를 활용할 때마다 'similar queries'로 부하가 발생한다.~~ <br>**2. 반복문 내 filter 사용으로 반복문의 범위가 커질 때마다 동일 쿼리가 해당 개수만큼 발생한다.**<br>

### 사용근거와 원인분석

학생들의 특정기간동안의 도서관 이용현황이 알고 싶어졌다.

```python
class Student(models.Model):
    name=models.CharField(max_length=256, verbose_name="학생명")
    grade_group=models.ForeignKey("group.GradeGroup", verbose_name="학급,반")
    instructor=models.ManyToManyField("teacher.Teacher", verbose_name="수업강사")
    ...
```

<br>

```python
class BookRent(models.Model):
    bookname=models.CharField(max_length=256, verbose_name="책명")
    student_id=models.ForeignKey("student.Student", verbose_name="학생")
    tstamp = models.DateTimeField(auto_now_add=True, verbose_name="대여날짜/시간")
    ...
```

<br>

```python
class Book(models.Model):
    bookname=models.CharField(max_length=256, verbose_name="책명")
    ...
```

지난편에 활용했던 예제로 활용해본다면 학생테이블(Student), 도서관대여테이블(BookRent)이 각각 있다고 해보자<br>
여기서 특정기간동안 **책별**로 **학생들이 이용한 횟수**를 알고 싶다. 그렇다면 view에서는 이렇게 작성될 수 있다.<br>

```python
...
book=Book.objects.all()

for b in book:
    book_used_count= BookRent.objects.filter(
        bookname=b.bookname,
        tstamp__date__gte=context["시작일"],
        tstamp__date__lte=context["종료일"],
    ).count()
    ...
    #나머지 처리
...
```

모든 책의 정보를 가져오고 반복문을 돌기 시작해 해당하는 책의 대여정보를 들고와서 counting을 해주는 모습이다. 돌아가는데는 문제가 없지만 한가지 문제에 직면했다. 책이 한,두권이면 상관이 없겠다만 만일 1000권이라면 1000+a번의 DB조회가 일어나게 된다. 실제로도 for문의 반복횟수가 많아질수록 완료되는 시간이 비례하여 오래걸린다.

### 첫번째 해결시도 (cached_property)

현재의 문제가 동일한 쿼리문이 반복적으로 동작하여 성능저하가 일어나고 있다는 점이다. 이러한 반복을 줄이기 위해 알아본 방법중에 **'cached_property'** 데코레이터를 활용해 보았다.<br>
해당 기능을 사용하여 최초호출 때 결과값을 캐싱해두어 다음 참조부터는 캐싱된 결과를 토대로 리턴하게 된다. 이를 통해 반복 호출 되는 문제를 해결할 수 있을 것이라 기대할 수 있다.<br><br>
model

```python
class BookRent(models.Model):
    bookname=models.CharField(max_length=256, verbose_name="책명")
    student_id=models.ForeignKey("student.Student", verbose_name="학생")
    tstamp = models.DateTimeField(auto_now_add=True, verbose_name="대여날짜/시간")
    ...

    @cached_property
    def get_book_count(self):
        return self.objects.filter(bookname=self.bookname).count()
```

view

```python
...
book=Book.objects.all()

for b in book:
    book_used_count= BookRent.get_book_count()
    ...
    #나머지 처리
...
```

위의 코드는 문법적으로 이상은 없지만 이 문제를 해결하기 위한 방법이 될 수 없음을 확인할 수 있다.<br>

1. 특정기간의 조건(시작일, 종료일)을 받지 못하고 있다.
2. 원하는 조건의 값이 반환되지 못한다.
3. 성능저하 문제의 해결이 될 수 없다.
   <br>

처음엔 cached_property가 매서드라고 생각을 해서 책제목,시작일,종료일을 매개변수로 전달할 생각이었다. 하지만 이것은 매서드가 아닌 클래스내 속성으로 동작하기 때문에 매개변수를 받을 수가 없다. (객체지향 무지 이슈 ㅠㅠ) 못쓰는 이유에 대해만 서술 했는데 캐싱에 대해 이해하고 싶다는 생각을 했다.

### 두번째 해결시도 (그룹화)

쿼리문을 반복문 밖에서 사용할 수 있지 않을까라는 생각을 해보았다. 결국엔 BookRent에서 책별로 count만 되면 해결되는 것아니 방법을 찾다가 **values**를 활용해 보았다.
<br><br>
view

```python
BookRent.object.filter(...).values("bookname")
```

```
<QuerySet [
    {'bookname':'칼퇴하는 일잘러의 업무 스킬, 파이썬 업무 자동화'},
    {'bookname':'일 잘하는 평사원의 업무 자동화'},
    ...
]>
```

원래 values()는 특정 데이터만 추출해 전체검색으로 인한 오버헤드를 줄일 때 유용한 기능이다. 하지만 위의 쿼리셋 결과물을 보면 BookRent 테이블이 책이름별로 정리가 된 모습이다. 이렇게 된다면 책별로 count를 측정할 수 있다는 것이다.<br>

count를 만들기 위해서 **annotate**를 사용해 count 필드를 추가하였다.

```python
BookRent.object.filter(...).values("bookname").annotate(rent_count=Count('*'))
```

```
<QuerySet [
    {'bookname':'칼퇴하는 일잘러의 업무 스킬, 파이썬 업무 자동화','rent_count':32000},
    {'bookname':'일 잘하는 평사원의 업무 자동화','rent_count':30000},
    ...
]>
```

annotate는 쿼리셋에서 추가적인 계산 필드를 생성해주고 쿼리 결과에 포함시키도록 한다. DB 쿼리문을 작성할 때처럼 Count, Sum, Avg와 같은 집계함수를 사용할 수 있다. 결국엔 Count를 활용하여 책별로 집계된 rent_count 필드가 생성되었음을 확인할 수 있다. 이외에도 필드명 변경(F)에서 활용이 가능하다는데 해결하는데는 큰 의미가 없어 설명은 생략하겠다.<br>

마지막으로 집계된 데이터들 중 원하는 데이터를 빠르게 가져올 수 있을지 고민해보았다. 반복된 DB 조회는 없애기는 했으나 결국엔 데이터 조회는 계속되고 있다. 탐색 속도를 향상시키기 위해 쿼리셋 데이터를 딕셔너리로 변환해 써보기로 했다. 딕셔너리는 순차적인 탐색이 아닌 key를 통한 탐색으로 원하는 데이터를 가져올 수 있는(해싱) 장점이 있다.

view

```python
...
book=Book.objects.all()
rent_counts=BookRent.object.filter(...).values("bookname").annotate(rent_count=Count('*'))

#딕셔너리로 전환
rent_counts_dict = {q['bookname']:q['rent_count'] for q in rent_counts}

for b in book:
    #딕셔너리로 조회해 활용한 모습
    rent_count=rent_count_dict.get(b.bookname)
    ...
    #나머지 처리
...
```

### 정리

1. values()를 활용한 데이터 그룹화
2. annotate()를 활용한 원하는 집계 데이터 생성
3. 딕셔너리를 활용한 데이터탐색 효율 늘리기

## 마치며

---

쓰고 싶은 말은 많았는데 잘 정리가 되지 못한 것 같다. 그리고 위의 해결방안이 될 수 있는지도 아직은 확인이 필요하다.<br>
하지만 전 글에서의 과정과 이번 해결시도를 통해서 약 400개의 데이터에 대해 1360 query 조회 되던것이 9로 줄었다.<br>
단계를 거칠 때마다 조회가 줄어들고 로드되는 속도가 향상되는게 눈에 보이니 보람도 있고 재미도 있었다.
