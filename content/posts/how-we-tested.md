---
title: "How We Tested"
date: 2023-11-21T02:15:43-06:00
tags: []
draft: false
---


Knowing how the test layers all play together is one thing, but getting them all working together turned out to be a little bit more difficult.

First - the application we were testing required a few different styles of scenarios.  We had the obvious API tests, but this application also worked with not only external REST APIs, but also Redis, and AWS infrastructure as well.  We needed to be able to controll all of these things for our component tests, but also wanted to make it quick and easy to use.  MockServer was our tool of choice for the other REST APIs - it just works, and does what it says on the tin.  For AWS, we always use Moto to do our component testing.  It is so much faster and cheaper than spinning up and down real AWS infrastructure, and for everything we've used so far, it works perfectly. 

<!--more-->

The hardest part was the fact that the application responds to internal events, but _also_ has catch-up logic (for missed messages) that happens on startup that we needed to test.  My counterpart came up with the idea of creating a MVC endpoint in ASP.NET to restart the application, and it was super super slick - the endpoint would call `IApplicationLifetime.StopApplication`, and then we put a `while (true)` loop in the `main` that would restart the lifecycle if a special environment was set.  This meant that each one of our tests could, in theory, completely restart the app just using a REST endpoint. 



## Write out the scenario

Even though we aren't using a Behavioral-Driven Development framework like Cucumber (we did audition it but we didn't like the way of writing tests, and had concerns about the rest of our team using it), we did use concepts from Gherkin to write our tests.  

For each scenario, we wrote out a Given-When-Then scenario description that would help us to write the test code itself.  These turned into code comments in our Python test case codebase - each scenario became a UnitTest case, that ended up looking like this

```python
from tests import BaseTestCase, givens
from assertpy import assert_that

class Test_MyScenario(BaseTestCase):
    """
    Given
        A state of the world before the test
    When
        Some action happens
    Then 
        Data should be saved to the database
    """

    def given(self) -> Givens:
        return Givens()

    def when(self):
        pass
    
    def test_scenario(self):
        self.when()
        data = self.get_data()
        assert_that(data).is_not_empty()
```

The `given` method, in each of the test cases, would return the data needed to be able to set up the state of the world.  The `when` would then setup the action to happen, and the `test_scenario` method would get discovered by pytest, and run everything.

We built out a whole framework around our tests such that we can have one of two behaviors:
1. In `DEV_MODE` (set by an environment variable), the BaseTestCase `setUp` method would set up our givens - inserting any data that was needed to set up for our test, and restart the application using a REST endpoint that recycled the entire application.
1. In regular mode, a `setup_component_tests` script would run, generating all of our test data before the tests start, so we can more appopriately test things like start up behavior

The framework became an integral part of making this work - we had builders for various complex data elements, data access helpers, and a whole suite of a custom API around interacting with MockServer to help us both build and validate expectations.  

We ended up with 70 something component tests, which took about 3 weeks to completely write from nothing.  If I broke down where that time went, I would probably save about a week of it went to just writing the scenario text - discovering what needed to be tested and what it looked like to test that.  About a week and a half was the framework. I got to the point though, that the framework was robust enough that I could code a scenario (assuming I had the text of it) in about 15 minutes because we had enough helpers and framework to make it easy.


