---
title: "Mapstruct 클래스타입 변환 / N:M 관계 매핑 에러"
categories:
    - spring
    - error
---

### 문제 발생
---

Mapstruct는 Builder를 통해 손수 객체 간 변환을 할 필요 없이 간단하게 변환을 적용해 주는 라이브러리이다.

평소 코드를 작성하면서 Dto 와 Entity 간에 변환이 필요할 때면 mapstruct를 이용해서 작업을 진행했고 이번에도 mapstrurct를 통해 매핑을 하는데 에러가 발생했다.

DTO 필드가 많아서 문제가 발생한 코드를 하나 가져와보면, 아래 이너 클래스는 원시타입이 아니라 ProjectDto.Add 클래스 타입의 list를 필드로 갖고 있다.

```java
@Getter
public static class Post {
    private List<ProjectDto.Add> projects;
}
```
<br>

ProjectDto.Add 또한 내부에 클래스 타입의 List를 가지고 있다.

```java
@Data
public static class Add {
    private List<ProjectSkillStackDto.Add> projectSkillStacks;
}
```

찾아보니 타입이 다른 경우에 자동으로 생성되는 Impl 파일에서 변환이 되지 않는 문제라고 한다.

<br>

### 해결 과정
---


구글링을 통해 해결 방법을 찾아보고 GPT 형님의 도움도 받아봤지만 해결이 되지 않아서 Builder를 통해 손수 변환을 해야 하나.. 고민하던 찰나 '이런 문제가 생기면 mapstruct를 사용하는 모두가 손수 변환을 해주는 건가? mapstruct를 활용해서 해결할 수 있는 방법이 있지 않을까?'라는 생각이 들어 새로운 방향으로 접근해 보고 해결 방법을 알게 되었다.

바로 `@Mapper` 애너테이션에 `uses`를 적용해서 해당 mapper에서 다른 mapper를 활용하는 것이다. 그럼 Builder로 일일이 변환 작업을 해줄 필요 없이 자동으로 변환될 것이다.

다음과 같이 변환에 필요한 Mapper 들을 모두 적용시켜 주니까 정상적으로 동작했다.

```java
@Mapper(componentModel = "spring", uses = {
        ProjectMapper.class, CareerMapper.class, CustomSectionMapper.class
})
public interface CvMapper {
    Cv cvPostToCv(CvDto.Post post);
    Cv cvPatchToCv(CvDto.Patch patch);
    CvDto.Response cvToCvResponse(Cv cv);
}
```

<br>

아래는 활용된 mapper 중 하나이다.
```java
@Mapper(componentModel = "spring")
public interface CustomSectionMapper {
    CustomSection customSectionAddToCustomSection(CustomSectionDto.Add customSectionAdd);
    CustomSectionDto.Response customSectionToCustomSectionResponse(CustomSection customSection);
    List<CustomSection> customSectionAddsToCustomSections(List<CustomSectionDto.Add> customSectionAdds);
    List<CustomSectionDto.Response> customSectionsToCustomSectionResponses(List<CustomSection> customSections);
}
```

<br>

전에는 무턱대로 `@Mapper(componentModel = "spring")`만 설정하고 변환했지만 여러 문제가 발생할 수 있다는 것을 인지했다.

<br>

### 다중으로 연관된 엔티티에서 추가 문제 발생
---


하나의 에러를 해결하니까 이번엔 또 다른 문제가 발생했다.

Cv 엔티티와 SkillStack 엔티티는 N:M 관계에 있다. 그래서 중간에 매개체로 CvSkillStack을 만들어서 1:N, N:1의 관계로 풀어냈는데, 매핑하는 과정에서 SkillStack이 null 값으로 나오는 문제였다.

문제의 원인을 찾아서 CvMapperImpl(CvMapper 구현체)을 살펴보던 도중, CvSkillStack 객체에 skillStackId가 설정되지 않고 null 값이 발생하는 것을 발견했다.

따라서 중간 매개체의 컬럼을 매핑시켜주는 방법을 검색하고 해결 방법을 찾게 되었다.

```java
@Mapping(source = "skillStackId", target = "skillStack.skillStackId")
CvSkillStack cvSkillStackAddToCvSkillStack(CvSkillStackAddDto cvSkillStackAdd);
```

위 애너테이션을 보면 source와 target으로 skillStackId를 명시하고 있다. source는 매핑의 원본 필드를 지정하고, target은 매핑의 대상 필드를 지정하는 것이다. 

이렇게 설정하면 CvSkillStackAddDto 객체의 skillStackId 필드 값을 CvSkillStack 객체의 skillStack.skillStackId 필드에 매핑하게 된다.

즉 addDto 객체로부터 skillStackId 값을 가져와서 CvSkillStack 객체의 skillStack 필드의 skillStackId에 설정함으로써, MapperImpl이 skillStackId를 확인하고 설정하여 정상적으로 매핑이 이뤄지는 것이다.

<br>

```java
@Mapping(source = "skillStackId", target = "skillStack.skillStackId")
ProjectSkillStack projectSkillStackAddToProjectSkillStack(ProjectSkillStackAddDto projectSkillStackAdd);
@Mapping(source = "skillStack.skillStackId", target = "skillStackId")
@Mapping(source = "skillStack.skillName", target = "skillName")
ProjectSkillStackResponseDto projectSkillStackToProjectSkillStackResponse(ProjectSkillStack projectSkillStack);
```

비슷한 관계를 갖고 있는 엔티티들의 매핑 작업에도 위와 같이 설정해줬다.