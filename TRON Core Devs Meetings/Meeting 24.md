# Core Devs Community Call 24
### Meeting Date/Time: September 26th, 2024, 7:00-8:00 UTC
### Meeting Duration: 45 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/101)
### Agenda
* Introducing PowerMock to write unit tests for Java-tron

### Details
* Jake

  Hello everyone. Today we are having Core Devs Community Call 24. Welcome everyone. For today's meeting, we have only one topic: "Introducing PowerMock to write unit tests for Java-tron" submitted by Super.

  Hi Super, can you hear me? Please share your screen and let's start now.

* Super

  OK. Hi guys, I am going to explain why I would like to introduce PowerMock to write unit tests today.

  There are several advantages as follows. First, writing unit tests with Mock allows us to break the dependency between the test object and services. For example, some unit tests depend on the database startup or service startup corresponding to the API. If the mock method is used, the dependency on services or database startup can be removed, thereby reducing the memory occupied by services and saving a lot of resources.

  The second function is that it can replace service calls. For example, when we call a certain service, it can be written in the Mock way without specifically writing the service interface. In this way, the running speed of test cases will be faster. We can save a lot of service calls. For example, when calling service A, and A calls service B and service C, etc. If we make these services into Mock, it will save a lot of trouble.

  The third point is to improve test efficiency. For example, within a unit of time, you can run more tests and the speed can be faster. Java-tron has introduced the Mockito module. However, due to the insufficient power of Mockito, for example, it cannot mock static methods, private methods, and constructors. If Java-tron wants to increase the unit test coverage rate to 70% or more than 80%, only introducing Mockito is not enough. It cannot solve the need for Mock in many scenarios.

  After introducing PowerMock, we can use Mock to write unit tests. The coverage rate should be able to increase to 80% or even more. There is no problem that a certain function cannot be covered because it cannot be constructed. There are many reasons for not being able to construct. Its parameters are very complex and the dependent service is very complex. A large number of services need to be started to call this method. Parameter construction is very troublesome. If PowerMock is used, all problems can be easily solved.

  I will share my practice of using PowerMock to improve the unit test coverage rate of Java-tron during this period for your reference and communication. If you have any questions, we can discuss them later. If you want to use PowerMock, then you must reference its dependencies. Currently, it is the PowerMock 2.0 version. Then you also need to use the annotation RunWith(PowerMockRunner.class) to indicate that the test case is run using PowerMockRunner. This must be added, otherwise PowerMock cannot be used. Another annotation is PrepareForTest({Util.class}), which is used to add all classes that need to be tested. Usually, when testing a category, these two prerequisites need to be added.

  I have sorted out some relatively complex unit test cases written with PowerMock and share them with you.

  The first practice is that PowerMock can assign private member variables non-intrusively. For example, here, in the wallet class, there is a private variable, chainBaseManager, which has no `get` and `set` methods. It may be assigned through the constructor. In this case, testing will be more troublesome. If PowerMock is used, I can directly assign a value to it. Create a Mock object for chainBaseManager, and then directly call this Mock function.

  The second is to call private methods non-intrusively. For example, here, in the RuntimeImpl class, it has a private method setResultCode. If I want to write a unit test case for this private method, since it is private and cannot be called directly, reflection may be used, which will be more troublesome. If PowerMock is used, it can be solved with one line of code. According to the object I defined here, the private method can be called directly by using the function Whitebox.invokeMethod. If reflection mechanism is used, there will be much more code than here. PowerMock has many encapsulated methods like this, which is very convenient when writing.

  The third is to mock constructors. For example, here I have a class to be tested. The usage of its constructor is relatively complex, and it is very troublesome to write test case logic during testing. If PowerMock is used to write, here directly define this class, and then use the method whenNew to return a Mock object to test this class.

  Here are a few more examples. Similarly, mocking static methods and private methods is very simple and convenient when writing test cases. I recommend that you get familiar with the official wiki of PowerMock here. It has many powerful functions, and people who often write test cases will be very impressed.

  I encountered a problem during the practice process. There is an incompatibility issue between PowerMock and Jacoco. The reason is that a class is decorated in PrepareForTest. JaCoCo cannot perform coverage statistics in this class. Both of them implement the functions of statistical coverage and mocking static classes by modifying bytecode files when loading classes. JaCoCo inserts statistical code into the class when loading the class. However, when PowerMock uses the PrepareForTest annotation, it will re-read bytecode information from the class file when loading related classes, resulting in the modifications of JaCoCo being lost, so it cannot be statistically counted. The solution is to use the offline mode of JaCoCo. Look at me redefining the steps here. In the end, the path of the unit test coverage report is the same as before. Finally, just upload the coverage data.

  For example, after introducing PowerMock, when writing unit tests, no matter how complex the code method is, there can be corresponding relatively simple methods to implement. For example, the most complex is the manager test. If too many things are used and when running to a certain case, a lot of code needs to be written. If we construct a case, it is very complex. If the reporting method is used, it may be relatively simple. That's about it. Do you have any questions?

* Andy

  What are the disadvantages or compromises of PowerMock for Java-tron tests? I see that you only shared the advantages.

* Super

  The first is that the coverage statistics in not compatible with Jacoco. Although this has been solved, I don't think it is very sufficient. We still need to look at the specific situation of other modules later.

  The second is that it is based on Mockito. It can be understood as an extended version of Mockito. If Mockito has already been introduced, then PowerMock plays an icing on the cake role.

* Andy

  How is its performance compared to before? Have you done a comparison?

* Super

  Surely it is much better to use Mock. First, the dependency is removed. In terms of running speed, for example, the number of interfaces run per unit of time is obviously better than when running unit tests before. It is based on Mockito, but it has more powerful functions. However, if these functions are not used, there is not much effect. This depends on the specific situation.

* Andy

  I understand. Does mocking these static methods and private methods still use Java's Reflection or something else at the bottom?

* Super

  Yes, part of it uses Reflection, and part of it is bytecode manipulation. Basically, it can be understood that everything that Mockito cannot do but you want to use is added here. From the past unit test experience of Java-tron, for a case that was very troublesome to construct in the past and needed to write a unit test to run, PowerMock can solve it perfectly. There will be no case that cannot be solved.

* Andy

  OK, I understand.

* Super

  In addition, for newly written code in the future, a perfect 100% test coverage can also be achieved no matter how the code is written.

* Super

  Any more questions? I see that some newly added unit tests have begun to be written with Mockito. I have looked at it. Many things that Mockito cannot solve can be solved by PowerMock. So I think it is very necessary to introduce it.

* Andy

  Will there be a conflict if both are introduced to Java-tron?

* Super

  No, they have corresponding versions. Its bottom layer also calls Mockito in part, and the other part is to strengthen it by modifying bytecode again. Lucas, do you have any opinions or questions?

* Lucas

  No other questions for now.

  I suggest using Mock in the future and not depending on system startup. I basically use Mock for all my unit tests now.

* Super

  Literally, there is no problem using Mockito. If we purely use Mockito, the code will be more complex. For example, when calling a private method, I see that when using Mock to call a private method, it is calling a private method using reflection. If PowerMock is used to call, the code is very simple. It has encapsulated all the content of calling private methods. There is no need to reprocess those things again. Just focus on construction and configuration.

  I have looked at the newly written unit tests and can do this. For some of the things that tested results fluctuated before, are the plans to keep them in the future? Do you have any ideas to redo them all over again? Do you have this idea?

* Allen

  Is there a plan to refactor existing use cases with PowerMock later?

* Super

  At present, I am working on improving coverage. However, for existing content, in-depth discussion may be needed. If the Mockito plus Power method is used for rewriting, it can indeed reduce the memory consumption of unit tests and execution speed, which is very helpful for the current situation. I think this issue can be discussed again.

* Super

  I think we need to introduce the powerful content and functions of Power first, and then discuss in depth to see if refactoring is needed. If refactoring is done, the workload will be very large.

* Allen

  So I understand that this still needs to be discussed and there is currently no plan.

* Super

  That's right, currently only considering improving the unit test coverage rate.

* Allen

  OK, I understand.

* Super

  Any other questions? If not, I'll stop sharing here.

* Jake

  Does anyone else have any questions? If not, today's meeting is over here. The next Call is tentatively scheduled for October 17. After that, we will adjust the development progress within the community in October according to the situation. Do you think it's okay?

* Murphy

  Sure, 17th it is. If not in time, it could be postponed by one week to the 24th. Let's discuss this in the community.

* Jake

  OK, that's it for today. Thank you all for attending. Goodbye!

 

### Attendance
* Ray
* Brown
* Super
* Andy
* Allen
* Lucas
* Murphy
* Jake