# Chapter- 3: Test Fixtures

## 3.0: Background

Generally, the unit tests consists of multiple tests where one needs a similar set of preconditions / arrangements are required to run the tests. For example, consider we want to write a class whose value can be incremented. To properly check the functionality of the class, we need to write multiple tests with multiple values. We can write the class as

```
class MyClass {

	int baseValue;

public:

	MyClass(int _baseValue) : baseValue(_baseValue) {}

	void Increment(int increment_by) {

		baseValue += increment_by;

	}

	int getValue() { return baseValue; }

};

```
Let's write a couple of GTests for validating the increment, i.e. increment_by_5 and increment_by_10 and the test cases could be written as

```
TEST(FirstTest, Increment_by_5) {

  // Arrange

	MyClass mc(100);

	int increment = 5;

  // Act

	mc.Increment(increment);

  // Assert

	EXPECT_EQ(mc.getValue(), 105);

}

TEST(FirstTest, Increment_by_10) {

  // Arrange

	MyClass mc(100);

	int increment = 10;

  // Act

	mc.Increment(increment);

  // Assert

	EXPECT_EQ(mc.getValue(), 110);

}
```
In both the test cases above, most of the "Arrange" parts (i.e creating the instance of MyClass by passing the value 100) are common which need to be repeated in every test.

To solve this problem Google Test provides a mechanism what we call as "Test Fixtures".

## 3.1: Test Fixtures : Creation and Usage

"Test Fixtures" in google test are a place where we can write common code which shall be executed every time a test is executed without writing it again and again. "Test Fixtures" are created using a class / struct which is derived from testing:: Test`. The common code in "Test Fixutures" which should be executed at the beginning and at the end of the tests are written in functions "SetUp ()" and "TearDown ()"

To use the "Test Fixtures", it's used in place of TEST name and TEST(s) with fixtures are used with macro "TEST_F". Here is how we'd write a test fixture for `MyClass`

```

struct  MyClassTest : public testing::Test {

	MyClass *mc;

	void SetUp() {

		mc = new MyClass(100);

	}

	void TearDown() {

		delete mc;

	}

};

```
and the tests could be written using fixtures as



```

TEST_F(MyClassTest, Increment_by_5) {

	mc->Increment(5);

	EXPECT_EQ(mc->getValue(), 105);

}

TEST_F(MyClassTest, Increment_by_10) {

	mc->Increment(10);

	EXPECT_EQ(mc->getValue(), 110);

}

```
The creation of `MyClassTest`, calling of `SetUp(..)` and `TearDown(...)` function happens automatically at the beginning and at the end of each tests without explicit call. so this is the way "Test Fixtures" helps us in writing the Tests.

To better undertand, how it's useful, let's write a full fledged `Stack` in C++ and then using the "Test Fixtures" to test the `Stack`.

## 3.2: Testing a full-fledged Stack with Test Fixtures

To further demonstrate the "Test Fixtures", let me write a simplified `class Stack` with stack options like `Push(...)` and `Pop(...)`. Here is how the code under tests look like. I've used `vector<>` for simplicity and reducing the code footprints

```

class Stack {

	vector<int> vstack = {};

public:

	void push(int & value) { vstack.push_back(value); }

	int pop() {

		if (vstack.size() > 0) {

			int value = vstack.back();

			vstack.pop_back();

			return value;

		}

		else {

			return -1;

		}

	}

	int size() { return vstack.size(); }

};

```

To be able to write Test Cases, we need something common over here too. As each and every test will be checking the `Stack`, the instance of `Stack` shall be created for each test case with sample data. This can be done by creating "Test Fixtures" of `Stack` class as shown below.

```

struct stackTest : public ::testing::Test {

	Stack s1;

	virtual void SetUp() {

		cout << "Setting UP..." << endl;

		int values[] = { 1,2,3,4,5,6,7,8,9 };

		for (auto &val : values)

			s1.push(val);

	}

	virtual void TearDown() {

		cout << "Tearning Down..." << endl;

	}

};

```
Once we have the fixture, let's create some of the tests using these fixtures.

### 3.2.1: Stack Pop(..) Test

This test checks whether of not items are popped in current order from the `Stack`. Here is how the test with Test Fixtures would look like


```

TEST_F(stackTest, PopTest) {

  // Arrange

  int lastPoppedValue = 9;

  // Act + Assert

	while(lastPoppedValue != 1)

		ASSERT_EQ(s1.pop(), lastPoppedValue--);

}



```

This is one case where the choice between non fatal 'EXPECT_*' and fatal 'ASSERT_*' is clear.

In this particular test case we should always use 'ASSERT_*' so as to stop at the first failure. It doesn't make sense to continue the test with 'EXPECT_*' because first failure may lead to all subsequent failures, hence the first failure is what we need to find out and fix.



### 3.2.3: Stack SizeValidity(..) Test

In this particular test case, we'll validate the size of the `Stack` by popping all elements. This ensures that the size is not misplaced.

```

TEST_F(stackTest, SizeValidityTest) {

	// Arranage

	int val = s1.size();

	// Act and Assert

	for (val; val > 0 ; val--)

		ASSERT_NE(s1.pop(), -1);

}



```

In this case also, we should use 'ASSERT_*' for the same reasons mentioned in 3.2.1


### 3.2.3: Stack MassPush(..) Test

This test is intended to push multiple data onto the Stack and then to verify that it's inserted in the correct order. Here is how the test cases would be written

```

TEST_F(stackTest, MassPushTest) {

	// Arranage & Act

	int ctr;

	for (ctr = 100; ctr < 10000; ctr += 100)

		s1.push(ctr);



	// Assert

	for (int val = (ctr - 100); val > 0; val -= 100)

		ASSERT_EQ(s1.pop(), val) << "!!!!!!!"<<endl;

}

```



## 3.4: Conclusion

In real life, most of the code will require multiple test cases which will have similar prerequisite. In these scenarios, "Test Fixtures" not only makes our lives not only by removing lines of code, but also by making sure that there is no duplication of codes.



