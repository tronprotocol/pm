# Core Devs Community Call 29
### Meeting Date/Time: December 19th, 2024, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/107)
### Agenda
* Upgrade Mockito and Replace PowerMock
* [Optimize System.exit Usage](https://github.com/tronprotocol/java-tron/issues/6125)

### Details

* Jake
  
  Hi everyone, we’ve got a couple of newcomers from the community today, and we have two topics to cover. One is Upgrade Mockito and Replace PowerMock, and the other is Optimize System.exit Usage.
  
  Super, are you ready to share with us?

* Super

  No problem, I’ll go now. Can you see my screen?
  
* Jake

  You're good to go.

**Upgrade Mockito and Replace PowerMock**

* Super

  So here’s the background on this topic: When we were using PowerMock, it was really convenient to mock static methods and constructors using `mockStatic` and `whenNew`. However, after upgrading the JDK, I found that PowerMock is no longer being maintained and doesn’t support JDK 17. After doing some research, I found that Mockito can replace PowerMock.

  This issue is about upgrading to a higher version of Mockito, specifically using `Mockito.mockStatic` and `Mockito.mockConstruction` to meet our needs, including constructor mocking.
  
  Looking into it, Mockito’s advanced versions, 4.0 and above, are split into two modules: mockito-core and mockito-inline. The core module covers most of the common Java testing scenarios, and it’s suitable for mocking regular classes and interfaces. If there are no special requirements, just using mockito-core should work fine for us.

  On the other hand, the inline module is for more special cases, like mocking static methods, constructors, and final classes.One thing to note: starting from version 5.0, the inline features have been merged into the core module, so for Java-tron, the core module alone can meet our requirements. However, version 5.0 only supports JDK 11 and above, and we still need to support JDK 8 in our project, so we can’t use version 5.0 directly. That’s why this issue is based on Mockito 4.0.
  
  Boson, do you have any thoughts on this?

* Boson

  No issues here. I was planning to upgrade PowerMock as part of the JDK 17 upgrade anyway, so since you’ve already looked into this, I can focus on other tasks.

* Super

  Great, I’ve already done some testing. Although Mockito can replace PowerMock, it’s not super friendly for mocking private methods. Mockito doesn’t support private methods directly, so we’d have to use reflection to call them. Alternatively, we can rethink the approach. Usually, private methods have a public entry point, and private variables can be set via reflection. Even with the upgraded version of Mockito, it doesn’t fully support private methods. But for mockStatic, mockConstruction, and final class support, it works well.
  
  One concern is that if Java-tron uses Mockito’s inline module, it could impact overall performance because it involves bytecode manipulation. I ran some tests, and it wasn’t too bad—running 2120 unit tests went from 22 minutes to just over 23 minutes, which is still acceptable. Test coverage also improved to over 70%, adding 700+ lines of code coverage based on the pull request I submitted.

  To summarize the versioning: Mockito 4.0 supports JDK 8 to JDK 17, while version 5.0 only supports JDK 11 and above. All the discussion above is based on Mockito 4.0. Any questions from anyone?
  
* Boson

  No other questions for now. After this feature is live, we’ll probably stop using Spring-based test startup, right?

* Super

  Yes. There are two aspects to consider here: one is service startup. We’ll need to improve this since many places in the code start services, and the ports they use clash, causing some services to fail to start. The other aspect is with databases, many test cases use `openDatabase`, and if we use mocks instead, the performance would improve significantly, making the tests run faster.

  Since there’s too much legacy code to modify, it’ll be more of an ongoing process, with people adjusting as they go. But I think around 24 minutes for the tests is still acceptable.

* Boson

  Is there a plan to improve CI performance and bring the build time under 15 minutes?

* Super

  That would go hand-in-hand with the parallel execution of unit tests we discussed earlier.

* Boson

  Here’s the thing: there are about 2000 tests now, and a lot of them are repetitive. For example, many database-related tests are covered across different services, so removing those redundant tests could help lower the execution time.

* Super

  The redundancy issue isn’t fully clear yet. After the Mockito upgrade, many of the test cases will need some adjustments. But as long as we keep parallel execution, we should be able to bring the time down to around 15 minutes. If we also remove the duplicate tests, we should be able to keep it under 15 minutes.

  By the way, you mentioned there’s an issue with test run times being really long on Mac OS, what was that about?

* Boson

  That issue only happens with Mac OS 12. After upgrading to version 14 or 15, it’s fine.

* Super

  Alright, so it’s not related to our current work. This feature should be included in version 4.8.0. I’ve already submitted the PR. If there are no more questions, I’ll wrap up my part.

* Boson

  Okay, I’ll start sharing then.

* Jake

  Sounds good!

**Optimize System.exit Usage**

* Boson

  My topic is pretty simple. It's about optimizing the use of `System.exit` in the system. Currently, there are several cases where `System.exit` is called directly to shut down the system after encountering specific exceptions. While this approach is straightforward, it leads to a few technical debts:
  1. Poor maintainability: Scattered `System.exit` calls make it hard to trace and maintain the program flow.
  2. Testing issues: Code with `System.exit` is hard to unit test because it causes Gradle tests to unexpectedly terminate, which messes up code coverage stats and reduces the reliability and stability of CI.
  3. Resource cleanup risks: `System.exit` doesn’t run finally blocks, meaning resources like database connections or file handles might not get closed properly.
 
  Right now in TRON, there are a dozen places where `System.exit` is used to exit the system. The problem with this command is that it doesn’t collect any information and exits immediately, so it’s difficult to trace the exit reason in the stack trace.

  The improvement I propose is to replace `System.exit` with throwing an error instead. So, whenever the system would have called `System.exit`, it should throw an error instead. Then, at the top level, we can have a unified exit logic based on that error. This makes it much easier to test the business logic.
  
  In terms of design, I suggest using errors instead of exceptions. For example, we could have a `TronError` as the base class with several subclasses based on the places where we’ve identified the need to exit. Some examples are `ConfigurationError`, `DatabaseError`, `EventSubscribeError`, and `OtherError`. Once we have these errors categorized, we can throw the appropriate error at the places where we previously had `System.exit`, and then exit from there.
  
  Previously, when we encountered `System.exit` in tests, we would use a JAR to intercept it. But this JAR is no longer supported in JDK 17, so solving the `System.exit` problem is essential if we want to upgrade to JDK 17.
  
  After making these changes, there are a few exceptions we need to be aware of. The first is when argument parsing fails, such as during SR node startup with keystore file. In the old setup, the main process would exit, but now we throw a `ConfigurationError` and then exit, so this behavior is consistent with before. We’ll need to track this further.
  
  Another issue happens during Spring initialization when errors are thrown, like `ConfigurationError` or `DatabaseError`, and it leaves two leftover processes called Thread-3 and leveldb. We haven’t pinpointed the exact cause of this yet, so we’ll need to track it as well.

  That’s about it. Any questions?

* Andy

  So, your solution is essentially wrapping all those scattered `System.exit` calls into different error types. You throw an error, and instead of immediately handling it, you let it propagate to the top level where you have a unified exit logic, right? And where exactly does the exit logic happen?

* Boson

  Yes, that’s correct. There’s a global thread handler for handling exits, kind of like a listener. If we don’t use that, we’ll need to design a new global handler for the errors.

* Andy

  I understand now that you're handling `System.exit`, which isn't part of the business logic, at the top level. You don’t need to include any business-specific details, just check for the error and exit accordingly.

* Boson

  Right, there won’t be any new logic. We’ll just change where the exit happens, but the exit itself remains the same. The exit will still occur, but now it will happen at the top level if it’s not handled by any lower layers. However, as I mentioned, there are still two threads `Thread-3` and `leveldb` created during Spring initialization that are causing issues when trying to exit. We still need to figure out the cause of those.

* Ray

  Are you concerned about those two threads because they’re preventing a clean exit?

* Boson

  Yes, that’s the issue.

* Ray

  Earlier, you were talking about throwing errors and then handling the exit logic yourself, but I was wondering if you’re still relying on the JVM’s exception handling to exit automatically? I’m not sure which option you’re going with. If you choose the first one, where you handle the exit yourself, would the daemon threads still cause issues?

* Boson

  Yes, I’m still proposing that we handle the exit manually. The issue is that the leftover daemon threads prevent a clean exit, so we’ll still need to use a similar approach to the one we have now to handle the shutdown process.
  
* Lucas

  So, the plan is to throw errors up to the top level and let that handle the exit? How do you ensure that the top-level entry point is in the right place?

* Boson

  I’ve traced it in the stack trace. I’ll list all the places where this happens in the issue for everyone to review and check for any mistakes. For example, during the startup phase, we throw the error directly in the main method. In other cases, like transaction processing, the logic is divided into layers like the interface layer, P2P layer, and repush layer. Block processing also has various threads for block production, syncing blocks, and processing messages via P2P.

* Lucas

  So, Spring initialization errors will be thrown to main, right?

* Boson

  Yes, exactly. Since Spring is running in the main thread, any errors there will go directly to main. Other errors will be thrown to their respective layers or threads.

* Ray

  When you throw errors to the top level, what exactly do you mean by "top level"? Is there a chance that these errors could be caught elsewhere before they reach the top level?

* Boson

  I’ve identified the top level based on the stack trace, it’s where the thread starts, so it’s a clear entry point for the errors.

* Lucas

  If you stick with the original `System.exit` approach, there wouldn’t really be any significant impact, right?

* Boson

  Correct, there wouldn’t be any immediate impact. The real issue is that this approach is a bad coding practice. We shouldn’t be exiting directly in the middle of business logic. Instead, errors should be propagated up the stack.

* Lucas

  Would it be better to define a new enum instead of using multiple error subclasses? I think defining 12 different error types is a bit much. We could just define a single type or description for each case, so instead of 12 subclasses, we’d have 12 different descriptions.

* Boson

  What do you all think?

* Andy

  You could include the exit code and description in the enum. That might simplify things.

* Boson

  That’s a good point. We don’t need to define so many error types. Let's stick with the plan: Step 1, throw errors to the top level; Step 2, during the next developer meeting, we can discuss whether we really need to exit at those points in the code.

  The root cause of this issue is really poor coding practices. Directly exiting within threads is just not a good design. We shouldn’t be calling `System.exit` in the middle of business logic, that is, it should be thrown up to higher levels.
  
  One of the teams from community has already pointed this out and requested that the fix be submitted to the dev branch as soon as possible.

* Jake

  So there's no question about whether we should do this, it needs to be done, and we need to get it live in the next release, right?

* Boson

  Exactly. Any other questions?
  
* Jake

  No? Then we’ll wrap up here. Thanks, everyone!


  
### Attendance
* Brown
* Andy
* Sunny
* Federico
* Allen
* Boson
* Daniel
* Lucas
* Ray
* SunnyBella
* Aaron
* Super
* Murphy
* Jake