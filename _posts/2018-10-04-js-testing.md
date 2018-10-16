---
layout: post
title:  "Javascript Projects Testing"
date:   2018-10-04 12:00:00 -0400
categories: software testing js javascript jest enzyme
author: Tania Paiva
---

Should I test this? the answer is a BIG YES. Tests are your friends, and saviors sometimes (often).

At Forward Financing we feel a big proud when our projects reach high test coverage, because thats our  
ensurance for making improvements and upgrades later with the warranty that existing functionality will  
keep working.

Mistakes can happen, always, and that is where tests can be real saviors, as an alert that  
something was altered by mistake, or that some functionality is not providing the expected result.

Tests verifies that the application meets the requeriments, it guarantees the code makes what we expect  
it to do.

Okay, tests can do a lot for us, but is not always sun and butterflies, writing good tests takes  
time and it gets better with some experience.

![Coverage]({{"/assets/test_coverage.png" | absolute_url}})

In this article we will talk about the testing tools we use for our javascript projects.

* [Jest](https://jestjs.io/)
Delightful JavaScript Testing, Jest is used by Facebook to test all JavaScript code including  
React applications. One of Jest's philosophies is to provide an integrated "zero-configuration" experience. 

* [Enzyme](https://airbnb.io/enzyme/)
Is a JavaScript Testing utility for React that makes it easier to assert, manipulate, and traverse your  
React Components' output.

Ok, those are cool names but what can we do with them?  
Here are some examples on how to use these tools:

We have a utility function that validates dates are in the following format DD/MM/YYYY

``` javascript
function areInvalidDates(values){
    let status = [];
    if (values.length > 0) {
      status = values.map(value => {
        if (value.split("/").length !== 3 ||
            value.split("/")[2].length !== 4 ||
            value.split("/")[1].length !== 2 ||
            value.split("/")[0].length !== 2
            ) {
            return 'invalid';
        } else {
          return 'valid';
        }
      });
    }
    return status.includes('invalid');
  }

  // the test
  describe('utils', function() {
    it('should have a function that validates an array of date string values', function () {
      const valuesWithInvalid = ["01/02/1990", "01/04/22", "02/02/2000"];
      const hasOneInvalidYear = utils.areInvalidDates(valuesWithInvalid);
      expect(hasOneInvalidYear).toBe.true;

      const valuesWithAnotherInvalid = ["01/02/1990", "01/4/2000", "02/02/2000"];
      const hasInvalidDay = utils.areInvalidDates(valuesWithAnotherInvalid);
      expect(hasInvalidDay).toBe.true;

      const valuesWithSomeInvalid = ["1/02/1990", "01/04/2000", "02/02/2000"];
      const hasInvalidMonth = utils.areInvalidDates(valuesWithSomeInvalid);
      expect(hasInvalidMonth).toBe.true;

      const valuesWithAllValid = ["01/02/1990", "01/04/2001", "02/02/2000"];
      const allValid = utils.areInvalidDates(valuesWithAllValid);
      expect(allValid).toBe.false;
    });
  });
```
