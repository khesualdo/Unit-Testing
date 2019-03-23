# Unit-Testing

### `Strict` vs. `Loose`

Strict mocking means that we must set up expectations (`Verify`, `Returns`) on all members of a mock object otherwise an exception is thrown. Loose mocking on the other hand does not require explicit expectations on all class members, instead default values are returned when no expectation is explicitly declared.

Default behaviour in Moq is loose mocking.

```C#
interface IGuidUtility

  CreateGuid();
  DeleteGuid();
  
class Utility

  IGuidUtility _guidUtility;
  
  void DoSomething(){
    _guidUtility.CreateGuid();
    
    ...
    
    _guidUtility.DeleteGuid();
  }

// This test will pass in loose mode
// This test will fail in strict mode, because we did not set up expectations for all members of the `moq`
// Specifically, we did not setup expectation for `DeleteGuid()`
[Test]
moq = new Mock<IGuidUtility>(MockBehavior.Strict);
utility = new Utility(moq.Object);
moq.Setup(x => x.CreateGuid()).Returns(...);

// This test will pass in loose mode
// This test will pass in strict mode, because we setup expectations for all members of the `moq`
[Test]
moq = new Mock<IGuidUtility>(MockBehavior.Strict);
utility = new Utility(moq.Object);
moq.Setup(x => x.CreateGuid()).Returns(...);
moq.Setup(x => x.DeleteGuid()).Returns(...);
```

### `Setup`

#### High-level Steps
1. Create an instance of Moq class that takes a interface or class with virtual methods as a generic parameter
2. Tell the Mock what you want to happen, specifying parameters, and even return values
3. Get back the mocked object which will behave in the way in which you told it to. Use this mocked object in your code.
4. Call `Verify` or `VerifyAll` on the mocked object to ensure that what you said would happen actually happened. This step is not always required.

#### Low-level Steps
```C#
// A either has to be an interface or an abstract class, 
// it may also be a regular class as long as B is a virtual method
Mock<A> mock = new Mock<A>(MockBehavior.Strict);

// B either has to be a method declared in an interface or an abstract class, 
// or be a virtual method in a regular class
// R is any valid object of return type of B
mock.Setup(m => m.B(...)).Returns(R);

// When B is called on mock.Object, R is returned
ValidateOpenIdConnectJSONWebToken(..., mock.Object, ...);
```

Such specific properties are required from `A` and `B`, because Moq will create its own version of `B` (for that it either needs to extend `A` and override `B` or implement `A` and define `B`) that follows the requirements set by the `Setup` and `Returns` method calls.

#### `It.IsAny<Guid>`
Allows any value of type Guid to be used as a parameter.

```C#
moq = new Mock<IUtility>(MockBehavior.Strict);
moq.Setup(m => m.CompareGuids(It.IsAny<Guid>, It.IsAny<Guid>, It.IsAny<Guid>)).Returns(True);

// Case 1 - Pass
moq.Object.CompareGuids(Guid.NewGuid(), Guid.NewGuid(), Guid.NewGuid());

// Case 2 - Pass
Guid sameGuid = Guid.NewGuid();
moq.Object.CompareGuids(sameGuid, sameGuid, sameGuid);

// Case 3 - Pass
moq.Object.CompareGuids(Guid.Empty, Guid.NewGuid(), sameGuid);
```

#### `It.Is<Guid>`
Restricts allowable parameters (of type Guid).

```C#
moq.Setup(m => m.CompareGuids(
  It.Is<Guid>(g => g.ToString().StartsWith("4"), 
  It.Is<Guid>(g => g != Guid.Empty), 
  It.Is<Guid>(g => g == expectedGuid))
).Returns(True);
```

#### Variable
Forces to pass the exact variable that was used for mocking.

```C#
Guid first = Guid.NewGuid();
Guid second = Guid.NewGuid();
Guid third = Guid.NewGuid();

moq.Setup(m => m.CompareGuids(first, second, third)).Returns(True);

// Case 1 - Pass
moq.Object.CompareGuids(first, second, third);

// Case 2 - Fail
moq.Object.CompareGuids(first, first, second);
```

### `Verify` vs. `VerifyAll` vs. `Assert`

`Verify` is used for verifying expected behaviour.
```C#
var mockFileWriter = new Mock<IFileWriter>();
useMockFileWriter(mockFileWriter.Object);

// Verify that `WriteLine` was called exactly 1 time on `mockFileWriter` in `useMockFileWriter()`
mockFileWriter.Verify(fw => fw.WriteLine("1001,10.53"), Times.Exactly(1));
```

`VerifyAll` will verify that all `Setup` calls were called (eg. in this case will check that `CopyLine` was called on `mockFileWriter` in `useMockFileWriter()`) as well as verify expected behaviour.
```C#
var mockFileWriter = new Mock<IFileWriter>();
mockFileWriter.Setup(fw => fw.CopyLine("1001,10.53")).Returns("1001,10.53");
mockFileWriter.Setup(fw => fw.WriteLine("1001,10.53")).AtMostOnce();
mockFileWriter.Setup(fw => fw.ReadLine()).AtMostOnce();
useMockFileWriter(mockFileWriter.Object);
mockFileWriter.VerifyAll();
```

`Assert` is used for verifying expected result.
```C#
var actual = DoSomething();
var expected = 1002;
Assert.AreEqual(expected, actual);
```
