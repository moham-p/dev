When writing unit tests in Java using Mockito, a common question pops up:

> “Should I use `@Mock` or `@Spy`?”

If you’ve asked that before, this guide will make sure you never forget the answer again.

---

## The Core Idea

When you're testing a class in isolation:

- Use `@Mock` for dependencies (like repositories, clients, etc.).
- Use `@InjectMocks` to inject those mocks into the class under test.
- Use `@Spy` when you want to call **real methods**, but override **some behavior** in the same class. Think of it as **partial mocking**.

---

## The Memory Trick

> **Mock = Mannequin**\
> **Spy = Secret Agent**

- A **mock** is like a mannequin — lifeless and fake, does exactly what you script it to.
- A **spy** is like a secret agent — it behaves like the real thing, but you can feed it secret instructions when needed.

With a `@Spy`, you get the real object **but with override powers**.

---

## Example: `@Mock` + `@InjectMocks`

### Service to be tested

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User getUserById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("User not found"));
    }
}
```

### Unit test

You're testing a `UserService` that depends on a `UserRepository`:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService; // real service, with mocked repository injected

    @Test
    void testGetUser() {
        Mockito.when(userRepository.findById(1L)).thenReturn(Optional.of(new User("John")));
        assertEquals("John", userService.getUserById(1L).getName());
    }
}
```

Use this when your service calls external dependencies and you want full control over them.

---

## Example: `@Spy` to Override Part of the Class

Let’s say your service looks like this:

```java
@Service
public class NotificationService {

    public String getMessageType(String userType) {
        // complex logic here
        return userType.equals("admin") ? "PRIORITY" : "STANDARD";
    }

    public String sendMessage(String userType, String message) {
        String type = getMessageType(userType);
        return "Sent " + type + " message: " + message;
    }
}
```

### Unit test

```java
@ExtendWith(MockitoExtension.class)
class NotificationServiceTest {

    @Spy
    private NotificationService notificationService;

    @Test
    void testSendMessage_withStubbedGetMessageType() {
        Mockito.doReturn("OVERRIDE").when(notificationService).getMessageType("guest");

        String result = notificationService.sendMessage("guest", "Hello!");
        Assertions.assertEquals("Sent OVERRIDE message: Hello!", result);
    }
}
```

Use `@Spy` when you're testing **real logic**, but want to **override** specific method calls.

---

## Spy + InjectMocks: Use With Caution

What if you want to **mock a dependency** (like a repository) **and** also **stub a method in your service**?

Technically, you can use `@Spy @InjectMocks` together:

```java
@Spy
@InjectMocks
private MyService myService;
```

But this isn't always reliable — especially when constructor injection is involved.

### Safer Approach: Manual Injection

```java
@ExtendWith(MockitoExtension.class)
class MyServiceTest {

    @Mock
    private MyRepository myRepository;

    @Spy
    private MyService myService = new MyService(myRepository);

    @Test
    void testSomething() {
        Mockito.doReturn("stubbed").when(myService).someInternalMethod();
        Mockito.when(myRepository.findSomething()).thenReturn("mocked result");

        String result = myService.mainMethod();
        Assertions.assertEquals("expected result", result);
    }
}
```

Or, fully manual with no annotations:

```java
@Test
void testWithManualSpy() {
    MyRepository mockRepo = Mockito.mock(MyRepository.class);
    MyService spyService = Mockito.spy(new MyService(mockRepo));

    Mockito.when(mockRepo.findSomething()).thenReturn("mocked");
    Mockito.doReturn("stubbedInternal").when(spyService).someInternalMethod();

    String result = spyService.mainMethod();
    Assertions.assertEquals("expected result", result);
}
```

This approach gives you full control and avoids surprises from framework behavior.

---

## Recap

- `@Mock`: Use for dependencies. It’s like a **mannequin**.
- `@Spy`: Use for real objects when you want to override specific methods. Like a **secret agent**.
- `@InjectMocks`: Injects your mocks or spies into the class under test.
- Always use `@ExtendWith(MockitoExtension.class)` when using Mockito with JUnit 5.

---

Just remember:

> **Mocks are mannequins. Spies are secret agents.**

---

Happy coding! 💻
