# Chapter- 4: GMock Usage, Tips and Tricks

In this chapter, we'll learn about the various usage of `GMock` and associated helper functions which may help us in writing our Mocks for C++ code. We'll start with the same coding example, via which we've learned about GMock in [Chapter- 3](https://github.com/9lean/Google_Test_Framework/tree/master/Chapter-%203).

## 4.0:  Mocking Without Deriving from the original class

In the last chapter, we wrote Mocks for the class

```
class DataBaseConnect {
public:
	virtual bool login(string username, string password)
	{ return true;}
	virtual bool logout(string username) { return true; }
	virtual int fetchRecord() { return -1; }
};

```
AS

```
class MockDB : public DataBaseConnect {
public:
	MOCK_METHOD0(fetchRecord, int());
	MOCK_METHOD1(logout, bool(string uname));
	MOCK_METHOD2(login, bool(string uname, string passwd));
};

```

However, it's not necessary to derive from the `DataBaseConnect` class and the `MockDB` class can simply reuse the name and be written as

```
// Note: This is a Mock DataBaseConnect
class DataBaseConnect {
public:
	MOCK_METHOD0(fetchRecord, int());
	MOCK_METHOD1(logout, bool(string uname));
	MOCK_METHOD2(login, bool(string uname, string passwd));
};

```
The name of the original class is borrowed so that it can be used seamlessly in the functions using the original class.
No changes are required in the `TEST(s)` if Mocks are not derived from the original class as the tests could be written as

```
TEST(MyDBTest, LoginAttemptNoDerivation) {
	// Arrange
	DataBaseConnect mockDb;
	MyDataBase db(mockDb);
	EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(Return(true));
	// Act
	int value = db.Init("Terminator", "I'll be Back");
	// Assert
	EXPECT_EQ(value, 1);
}

```

### 4.0.0: When should we use this ?

This kind of Mocking is useful when we want to mock non virtual and sometimes even private functions. However, its not the derivation is not required or useful. The basic idea behind derivations comes from the fact that we sometimes want to call the original function via the mocks as well as verify that the pure virtual is implemented by the mocking class.

## 4.1:  Invoking Original and Other Implementations from the Mocks

Google Mocks allows us with the mechanism of calling Original Implementation as well as other implementation Stubs. We do this by using `Invoke(...)` function in the actions.
Let's go back to original implementation where my `MockDB` is derived from `DataBaseConnect`.

```
class MockDB: DataBaseConnect {
public:
	MOCK_METHOD0(fetchRecord, int());
	MOCK_METHOD1(logout, bool(string uname));
	MOCK_METHOD2(login, bool(string uname, string passwd));
};

```
Let's also provide some implementation in the original class `login` function which we intend to call from the Mock class.

```
class DataBaseConnect {
public:
virtual bool login(string username, string password) {
    cout << "LOGIN ORIGINAL....." << endl;  return true;
  }
virtual bool logout(string username) { }
virtual int fetchRecords() {}
};

```

To call the orginal `login(...)` function, we need to create an instance of original `DataBaseConnect` class and include this in `Invoke`

```
using ::testing::Invoke;  // needed for calling Invoke

TEST(MyDBTest, LoginAttemptNoDerivation) {
	// Arrange
	MockDB mockDb;
	MyDataBase db(mockDb);
	DataBaseConnect originalDB;
	EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(Invoke(&originalDB, &DataBaseConnect::login));
	// Act
	int value = db.Init("Terminator", "I'll be Back");
	// Assert
	EXPECT_EQ(value, 1);
}

```
Similar things can be done by using `ON_CALL(...)`

```
TEST(MyDBTest, LoginAttemptNoDerivation) {
	// Arrange
	MockDB mockDb;
	MyDataBase db(mockDb);
	DataBaseConnect originalDB;
	ON_CALL(mockDb, login(_, _)).WillByDefault(Invoke(&originalDB, &DataBaseConnect::login));
	// Act
	int value = db.Init("Terminator", "I'll be Back");
	// Assert
	EXPECT_EQ(value, 1);
}

```

The `Invoke(...)` function can be used for calling any function not just the base class functions. Here is how we can do that.
Let's define a function whose signature matches with `DataBaseConnect::login` function in parameters as well as in return types are same.

```
struct TestABC {
	bool gFn(string a, string b) { cout << "Non Class Function" << endl; return true; }
};

```
If we change the `EXPECT_CALL` as

```
EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(Invoke(&aInstance, &TestABC::gFn));

```
This will trigger `TestABC::gFn` function to be called.

We can also call some global functions by using `InvokeWithoutArgs(..)`, but the global function should not take any parameters.

```
bool GlobalFn() {
	cout << "Global Function..." << endl;
	return true;
}

using ::testing::InvokeWithoutArgs;
EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(InvokeWithoutArgs(GlobalFn));

```

## 4.2: Setting Default() actions

Sometimes it's tiring to write actions on `EXPECT_CALL`, if we have to write more than 1 `EXPECT_CALL`. In those cases, we can set the expectation using `ON_CALL` and then use all the `EXPECT_CALL` to call the default function.

For example, we want the same Global Function `GlobalFn` to be called with when we call our Mock function for `login` and `logout`, we can do the same by calling `DoDefault()` function and changing the test as

```

TEST(MyDBTest, LoginAttemptNoDerivation) {
	// Arrange
	MockDB mockDb;
	MyDataBase db(mockDb);
	TestABC testDefault;

  // Setting the expectation
	ON_CALL(mockDb, login(_, _)).WillByDefault(Invoke(&testDefault, &TestABC::gFn));
	ON_CALL(mockDb, login(_, _)).WillByDefault(Invoke(&testDefault, &TestABC::SomeOtherFn));

  // Making Default Calls
	EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(DoDefault()); // calls gFn
	EXPECT_CALL(mockDb, logout(_)).Times(1).WillOnce(DoDefault());  // calls SomeOtherFn

	// call both login and logout function
	int value = db.Init("Terminator", "I'll be Back");
	db.close("Terminator");

	EXPECT_EQ(value, 1);
}
```

## 4.3: Doing Multiple Actions using DoAll(...)

Till now we've seen that only one action is called when we call a mock. Google Mock allows us to invoke Multiple actions when calling a mock if they satisfy the following conditions.

- Only the last action Return will be considered

- All actions, but the last actions should return `void`

Here is one example where we're not only calling the `TestABC` function, but also Returning `true` as the return value of Mock. We need to change the return type of the `gFn(..)` as `void`.

```
struct TestABC {
	void gFn(string a, string b) { cout << "TestABC Function" << endl; return; }
};

using ::testing::DoAll;

EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(DoAll(Invoke(&testDefault, &TestABC::gFn), Return(true)));

```
We can `Invoke` the same function twice

```
EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(DoAll(Invoke(&testDefault, &TestABC::gFn), Invoke(&testDefault, &TestABC::gFn), Return(true)));

```
We can even call a global function here as

```
EXPECT_CALL(mockDb, login(_, _)).Times(1).WillOnce(DoAll(Invoke(&testDefault, &TestABC::gFn),

		InvokeWithoutArgs(GlobalFn), Return(true)));

```

## 4.4: Conclusion
In this chapter we've seen some of the advanced topics on using Google Mock.



