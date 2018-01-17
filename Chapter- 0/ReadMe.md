# Chapter- 0: Introduction to Google Test Framework

Google Test is a unit testing framework for C++. This page is intended to demonstrate usage of Google Test for testing C++ code using practical examples.

## 0.0: Installation

The installation requirements and instructions could be found [Here](https://github.com/google/googletest)

A Google Test Primer can be found [Here](https://github.com/google/googletest/blob/master/googletest/docs/Primer.md)

Post installation, we need to link the gtest lirbraries and include ```<gtest.h>``` file in the code to start using Google Test


## 0.1: Writing the first Google Test

Google Test are initialized and triggered from a function. The main function is written as

```
int main(int argc, char **argv) {
  ::testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}
````

It will call all the tests written/included in the scope of the file where `RUN_ALL_TESTS()` is called.

To write Google Test, we'll have to use `TEST(,)` macro which takes two parameters. First parameter is the name of the test while the second parameter is the subtest. Test's are grouped together with the name of First Parmeter.

Here are the simple assertions

```
TEST(SimpleTest, SubTest_1) {

ASSERT_TRUE(1 == 1);

ASSERT_FALSE(1 == 2);

}

````

When we run the test, it will pass as both assertions are correct. There is a `NOEXPECT_*` version also available with Google Test and the same set of code could also be written as 


```
TEST(SimpleTest, SubTest_1) {

EXPECT_TRUE(1 == 1);

EXPECT_FALSE(1 == 2);

}

```

## 0.2: Fatal and Non-Fatal Assertions

Any assertion in google tests leads to 3 scenarios

- Success
- Non Fatal Failure
- Fatal Failure

### Non-Fatal Assertions

Are those which doesn't stop the execution of the test if the assertion fails. For example, in the code below

```
TEST(SimpleTest, SubTest_1) {

EXPECT_TRUE(1 == 2);

EXPECT_FALSE(1 == 2);

}

```

The assertion `EXPECT_FALSE(1 == 2)` will execute even if `EXPECT_TRUE(1 == 2)` fails.

### Fatal Assertions

Are those which stops the execution of the code at the first failure. For example, in the code below

```
TEST(SimpleTest, SubTest_1) {

ASSERT_TRUE(1 == 2);

ASSERT_FALSE(1 == 2);

}

````

The assertion `ASSERT_FALSE(1==2)` will not execute as the test `ASSERT_TRUE(1 == 2)` leads to a failure


### Differentiation

- Any assertion with `EXPECT_*` is **Non-Fatal Assertion**
- Any assertion with `ASSERT_*` is  **Fatal Assertion**


## 0.3: Comparision Assertions

Apart from `TRUE` and `FALSE`, there are many other binary comaprison asserts for both fatal(ASSERT_*) and non-fatal(EXPECT_*) failures. Here is the list of them

### List of Comparision Assertions 

The comparison assertions are used as comma separated values and not as an operation. For example `ASSERT_EQ(1,1) / EXPECT_EQ(1,1)`

`EXPECT_* / ASSERT_*`

- `EQ` - Equal
- `NE` - Not Equal
- `LT` - Less Than
- `GT` - Greater Than
- `LE` - Less Than Equal
- `GE` - Greater Than Equal


## 0.4: Conclusions

In this chapter, we learnt how to use the basic assertion of google tests and the difference between fatal and non-fatal assertions

