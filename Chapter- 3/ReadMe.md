# Chapter - 3 : Using Google Mock (The Mocking Framework)

Google Mock is a mocking framework to emulate the behaviour of API's and Interfaces in C++.

Mocks should not be compared with __Stubs__ or __Fakes__ as they are different than mocks. Both __Stubs__ and __Fakes__ can be used along with __Mocks__ to assist in the testing behaviour. Here is what we should keep in mind to properly understand these terms

## 3.0 Mocks, Fakes, and Stubs

#### Mocks

__Mocks__ are used for testing the behaviour of API(s) / Interfaces which will be used in component under test.. For example, we can _Mock_ the HTTP API(s) to test our component which is using HTTP for fetching the data from the server

#### Fakes

__Fakes__ are similar to working objects, but may take any shortcut to implement the behaviour. For example, it can create a dummy HTTP server which runs on localhost and provide random or pre selected response to the requests.

#### Stubs

__Stubs__ are implementation which provide predefined output of the actions. For example, for testing an HTTP client, it may return 200 OK all the time.

This chapter is dedicated to understand, create, and use the __Mocks__ as provided by Google Mocks.

## 3.1 Writing a Mock Database Connection API(s) / Interface(s)

Mocks are written not to test the API(s) / Interface(s) we are going to use, but to test our components which is going to use these API(s) / Interface(s).

To understand, let's consider the availability of a `class DataBaseConnect` API/Interface which allow the users to login, logout, and fetch records from the database. Here is how the class would look like

```
class DataBaseConnect {
public:
	virtual bool login(string username, string password) 
	{ return true;}
	virtual bool logout(string username) { return true; }
	virtual int fetchRecord() { return -1; }
};

```
Let's write a component which will use this `DataBaseConnect` to connect to a database.  Let's call it `MyDatabase` and will be written as 

```
class MyDatabase {
	DataBaseConnect  & dbConnect;
public:
	MyDatabase(DataBaseConnect & _dbC) : dbConnect(_dbC) {}
	int Init(string uname, string passwd) {
		if(dbConnect.login(uname, passwd) != true) {
			cout<<"Failed to connect >>>>> "<<endl;
			return -1;
		} else {
			cout<<"Successful Connection >>>>>"<<endl;
			return 1;
		}
	}
};

```
The `Init` function will use the database login function to connect the database using the user name and password.

Since we can't use actual implementation of `DataBaseConnect` class for the unit testing purpose, we'll have to write a __Mock__ for the same.  The __Mock__ is written by deriving from the class and generate dummy implementation using Google Mock.

We will use Google Mock macros `MOCK_METHOD (x)` where `x` represents the number of parameters a method will take. Here is how we will write the Mock code

```
class MockDB : public DataBaseConnect {
public:
	MOCK_METHOD0(fetchRecord, int());
	MOCK_METHOD1(logout, bool(string uname));
	MOCK_METHOD2(login, bool(string uname, string passwd));

};

```
Remember, we don't need to create any implementation of these methods. Google Test will do it automatically.

NOTE: The sequence of macro doesn't matter as long as we write the appropriate macro for each function defined in the interface class `DataBaseConnect`.

## 3.1 Writing Tests using Mock

Once we have the __Mock__ implementation, we need to write the tests. As we've seen in earlier chapters we write a unit tests as "Arrange - Act - Assert".  

The expected behaviour of the __Mock__ is defined in the "Arrange" Section and can be done using "EXPECT_CALL"  and "ON_CALL"

### 3.1.1 Using EXPECT_CALL

Here is how we write the test using `EXPECT_CALL`. 

```
TEST(MyDBTest, LoginTest) {
	// Arrange
	MockDB mdb;
	MyDatabase db(mdb);
	EXPECT_CALL(mdb, login("Terminator","I'm Back"))
	.Times(1)
	.WillOnce(Return(false));
	// Act
	int retValue = db.Init("Terminator", "I'm Back");
	// Assert
	EXPECT_EQ(retValue, 1);
}

```
The `EXPECT_CALL` sets that expectation of the function that it will be called at least once and shall return `true`. As we can see from the `TEST` above, we are providing the parameters to the `login` function which needs to be matched when the call is made. If we change these parameters that then the tests will fail. 
To avoid having static parameters in expect, we can use underscore `_`, which means the behavior login will not depend upon the parameter. The changed code shall look like

```
EXPECT_CALL(mdb, login(_,_)
.Times(1)
.WillOnce(Return(false));
```
Since the `Init` function checks two conditions, which means we can write a second test to test the implementation for login failures. Here is how we can write a login failure test

```
TEST(MyDBTest, LoginFailure){
	// Arrange
	MockDB mdb;
	MyDatabase md(mdb);

	EXPECT_CALL(mdb, login(_,_))
	.Times(1)
	.WillOnce(Return(false));

	// Act
	int retValue = md.Init("Terminator", "I'll be back");
	// Assert
	EXPECT_EQ(retValue, -1);
}
```
Now this test will pass as `login` is failed. However, we may have an implementation where we try to login twice in case of a failure. Let's rewrite the `Init` function to do the same.

```
int Init(string uname, string passwd) {
		if(dbConnect.login(uname, passwd) != true) {
			cout<<"Failed to connect >>>>> "<<endl;
			if(dbConnect.login(uname, passwd) != true) {
				cout<<"Failure 2nd Time.. Returning.."<<endl;
				return -1;
			}
		} else {
			cout<<"Successful Connection >>>>>"<<endl;
			return 1;
		}
	}
```
Now my 2nd test will fail because the `login` is called twice instead of once. We can change the expectation to get it called twice.

```
EXPECT_CALL(mdb, login(_,_))
	.Times(2)
	.WillOnce(Return(false));
```
That's how we can use the conditions to set the behaviour of the __Mocks__ 

### 3.1.2 Using ON_CALL

Similar to `EXPECT_CALL`, `ON_CALL` can also be used to set the expectation, but there is a difference, `ON_CALL` tells the test, what to do when the function is called, but doesn't enforce that the function must be called for the test to pass. Let's change our first test to with `ON_CALL` to demonstrate the same.

Let's change our first test to with `ON_CALL`  to demonstrate the same.

```
TEST(MyDBTest, LoginTest) {
	// Arrange
	MockDB mdb;
	MyDatabase db(mdb);
	ON_CALL(mdb, login(_,_))
	.WillByDefault(Return(true));
	// Act
	// Setting the value directly. Not calling the function
	int retValue = 1;  
	// Assert
	EXPECT_EQ(retValue, 1);
}
```
The test will pass which was not possible if we've used 'EXPECT_CALL`.

__So now the question is where to use the `ON_CALL` ?__

`ON_CALL (...)` can be used in situations where we have multiple functions to call, but we don't know which function will be called at runtime. To understand this point, let's add another logging function called `login2 ()` in the interface and mock it.

```
// The login function in DataBaseConnect class
virtual bool login2(string username, string password) 
	{ return true;}

// The corresponding Mock function in MockDB class
	MOCK_METHOD2(login2, bool(string uname, string passwd));

```
Now let's change the implementation of the `Init(...)` function so that either of the login function get called based on a random number being even or odd. Here is how we can write the code for the same.

```
int Init(string uname, string passwd) {
		//cout<<"RANDOM NUMBE IS "<<rand()<<endl;
		int randno = rand() % 2;
		if(randno == 0) {
			cout<<"EVEN RANDOM NUMBER CALLED>>>>>>>>>"<<endl;
			return dbConnect.login2(uname, passwd) == true;
		} else {
			cout<<"ODD RANDOM NUMBER CALLED>>>>>>>>>"<<endl;
			if(dbConnect.login(uname, passwd) != true) {
				cout<<"Failed to connect >>>>> "<<endl;
				if(dbConnect.login(uname, passwd) != true) {
					cout<<"Failure 2nd Time.. Returning.."<<endl;
					return -1;
				}
			} else {
				cout<<"Successful Connection >>>>>"<<endl;
				return 1;
			}
		}
	}
```
In this case, sometimes the `login(...)` function gets called while at times `login2(...) function gets called, but the expectations have been set by `ON_CALL` and its not possible to test this code using `EXPECT_CALL`.

## 3.2 : Conclusion 

In this chapter, we've learned how to use Google Mocks and set the expectations using `EXPECT_CALL` and `ON_CALL`.


