---
title: (Django) filter 올바르게 사용하기 1편
author: 쿠로
date: 2024-12-04
description: 쿠로 첫글 남깁니다.
---

최근에 django 업무로 다시 넘어오면서 발생했던 문제, 그것을 해결한 부분이 있어 글로 남깁니다.<br>
해당 편은 총 2편으로 제작될 예정이며, 지금의 조치가 올바르지 않을 수 있으나 효과가 있었기에 글로 남기게 되었습니다.<br>

## 개요

---

### 문제사항

1. DB에 저장된 데이터들을 가공하여 통계를 내는 과정에서 외래키로 연결된 필드를 활용할 때마다 'similar queries'로 부하가 발생한다.
2. 반복문 내 filter 사용으로 반복문의 범위가 커질 때마다 동일 쿼리가 해당 개수만큼 발생한다.

## 본론

---

### 사용근거와 원인분석

이 글에서는 1번의 원인/해결법을 다뤄볼 예정이다.
<br><br>
당연하게도 처음에는 Model.object.filter(...) 또는 all()을 사용할 때 그 즉시 쿼리문이 실행될 것이라 생각 해왔었다. 예시 모델 코드를 준비해 보았다.<br>

```
class Student(models.Model):
    name=models.CharField(max_length=256, verbose_name="학생명")
    grade_group=models.ForeignKey("group.GradeGroup", verbose_name="학급,반")
    instructor=models.ManyToManyField("teacher.Teacher", verbose_name="수업강사")
    ...
```

학생 클래스에서 '학급,반', '수업강사'가 각각 타 테이블과 연결이 되어 있다.<br>
아래 view에서는 이렇게 구현을 해보았다.

```
students=Student.objects.all()

for student in students:
    item = {
        'name':student.name,
        'grade_group':student.grade_group.grade, #학년별 반
        'instructor':student.instuctor.name, #강사 내역
        ...
    }
```

위처럼 작성할 경우 for문의 범위만큼 grade_group에서는 query가 발생하게 된다.<br>
학생이 100명이면 저 한줄 때문에 100개의 query가 발생된다는 것이다.<br>
어떻게 생각하면 당연하게도 저 로직이 실행이 된다면 student에 저장된 foreign key(id)를 들고 오게 되는데 우린 해당 테이블의 grade값을 알고 싶기 때문에 grade_group 테이블을 조회하는 query가 발생할 수 밖에 없는 것이다.<br>
또한 instructor은 manytomany 형태이기 때문에 위처럼 view가 작성될 경우 단일값이 아니기에 에러가 발생하게 된다.<br>

### 해결방안

for문 내에서 query가 발생되지 않도록 미리 조치를 할 필요가 있다.<br>
이를위해 django에서는 2가지 기능을 제공한다.(select_related, prefetch_related)<br>
파악한 바로는 두 기능 모두 미리 필요한 데이터를 가져오는데 사용되는 메서드이다. 위처럼 foreign_key로 연결된 경우 해당 테이블의 데이터를 미리 가져올 때 쓰기에 적절한 메서드라 생각했다.<br>

prefetch_related 메서드를 활용해 해결된 것을 확인하였는데 select_related와 무슨 차이가 무엇인지는 아직 이해하지 못했다.<br>

```
students=Student.objects.all().prefetch_related('grade_group','instructor')
```

view 코드에서 위처럼 수정을 해주었다.<br>
반복문이 동작하기 전에 foreign key의 테이블에 접근해 해당 테이블의 데이터를 가져올 수 있도록 하였다.<br>
실제 쿼리문은 아래처럼 작성된 것을 확인할 수 있었다.

```
SELECT *
FROM grade_group
WHERE grade_group.id IN (1,2,3,4,...);
```

student 테이블에서 등록된 grade_group의 id의 데이터 전부를 가져온 모습을 볼 수 있다.<br>
반복문에 들어서기 전 모든 데이터를 들고 왔기에 반복적인 query가 동작하지 않는 것을 확인할 수 있었다.

## 마치며

---

해결하며 많은 시간이 걸리지는 않았지만 아직도 django에 대해서는 빙산의 일각 정도의 활용도를 보여주고 있음을 느꼈다. 제공되는 메서드들을 외운다라기 보다는 많은 상황들을 접해보면서 '아 이런 것이 있었지' 생각하며 적용해보고, 안된다면 새로 개척하고 그것을 두려워하지 않으려 노력하는 자세가 되어야한다.(사실 아직 이점은 매우 어렵다.)<br>
알아보면서 느낀건데 차후에는 django가 lazy-loading 특성을 갖는 것이 무엇인지와 N+1문제가 계속언급되어 더 알아보고 싶은 마음이 들었다.<br>
다음편에서는 위에서 언급한 2번의 문제에 대해 실패했던 사항과 다른 방도로 해결한 사항에 대해 기술하겠다.
