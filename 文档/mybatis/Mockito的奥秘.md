```java
@Mock
private Person person;

@Test
public void testPerson() {
  when(person.getName()).thenReturn("jack");
  Assert.assertEquals("jack",person.getName());
}
```



```java
@CheckReturnValue
public static <T> OngoingStubbing<T> when(T methodCall) {
    return MOCKITO_CORE.when(methodCall);
}
```



```java
public <T> OngoingStubbing<T> when(T methodCall) {
    MockingProgress mockingProgress = mockingProgress();
    mockingProgress.stubbingStarted();
    @SuppressWarnings("unchecked")
    OngoingStubbing<T> stubbing = (OngoingStubbing<T>) mockingProgress.pullOngoingStubbing();
    if (stubbing == null) {
        mockingProgress.reset();
        throw missingMethodInvocation();
    }
    return stubbing;
}
```



```java
@Override
public OngoingStubbing<T> thenReturn(T value) {
    return thenAnswer(new Returns(value));
}
```