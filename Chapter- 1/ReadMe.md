# Chapter- 1: Writing Unit Tests (Arrange - Act - Assert )

Last chapter talked about basic assertions of Google Test Framework. In this chapter, we'll details about how to write unit test cases using the _Arrange-Act-Assert_ 

## 1.0: Arrange Act Assert

All unit tests should have the following 3 sets

- Arrange : Arranage all the preconditions needed to run the tests
- Act :  Run the test
- Assert : Verify the Test result (expected or otherwise)

Since all unit tests must be able to run independently, we need to arrange for preconditions to run the test as well as check the test results in the same test.


## 1.1: Arrange - Act - Assert Test

A Google Test with _Arrange - Act - Assert_ can be written as

```
TEST(FirstTest, SubTest_1) {
	// Arrange
	int value = 100;
	int increment = 5;

	// Act
	value = value + increment;

	// Assert
	ASSERT_EQ(value, 105)
}

````

## 1.2: Arrange Act Assert using a C++ Class

Let's write a C++ class called which takes a parameter call `id` as

```
class MyClass {
	string id;
public:
	MyClass(string _id) : id(_id) {}
	string GetId() { return id; }
}

```

We can write the test for the above class as 

```
TEST(ClassTest, SubTest_1) {
	// Arrange
	MyClass mc("root");

	// Act
	string value = mc.GetId();

	// Assert
	ASSERT_EQ(value, "root");
}
```

Depending upon the compiler we use, the above test can be successful or failure. The reason for the failure is that the `ASSERT_* / EXPECT_*` checks the binary value and to properly check the `string` we need to use string based asserts. The assert above should ideally be written as

```
ASSERT_STREQ(value.c_str(),"root");

``` 

## 1.3: String Assertions

Google Test provides specific string based assertions which must be used while comparing the strings for both non-fatal and fatal assertions. They ar

`ASSERT_STR* / EXPECT_STR*`

- `EQ` - Equal
- `NE` - Not Equal
- `CASEEQ` - Ignore case Equal
- `CASENE` - Ignore case Not Equal


## 1.4: Conclusions

In this chapter, we learnt how to use write unit tests using Arrange - Act - Assert and using string based asserts 

