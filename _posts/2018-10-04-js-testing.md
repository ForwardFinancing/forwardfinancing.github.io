---
layout: post
title:  "Javascript Projects Testing"
date:   2018-10-04 12:00:00 -0400
categories: software testing js javascript jest enzyme
author: Tania Paiva
---

Should I test this? the answer is a BIG YES. Tests are your friends, and saviors sometimes (often).

At Forward Financing, we feel proud when our projects reach high test coverage. It's our best insurance  
for making improvements and upgrades later with the warranty that existing functionality will  
keep working.

Mistakes can always happen, and that is where tests can be real saviors, as an alert that  
something was altered by mistake, or that some functionality is not providing the expected result.

Tests verify the application meets the requeriments, it guarantees the code makes what we expect  
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

This first example makes use of Jest library to set what we expect to see with each set of values.
The behavior of the function on each case is evaluated and we can set the expected result according  
the params.

Now how does Jest and Enzyme work with React components? :thinking:
Let's say we have a login box component with fields for email and password:

```javascript

class LoginBox extends Component {

  constructor() {
    super();
    this.state = {
      email: '',
      password: ''
    };
  }

  saveInput(name, event) {
    if ((event.keyCode || event.which) === 13) {
      this.submitForm(event);
      return;
    }
    let state = {};
    state[name] = event.target.value;
    this.setState(state);
  }

  submitForm(event) {
    event.preventDefault();
    this.props.onSubmit(this.state);
  }

  render() {
    return (
      <div className="login-box">
        <div className="login-box__header">
          <h1>Login</h1>
        </div>
        <div className="login-box__content">
          {
            this.props.error &&
            <div className="login-box__error">{this.props.error}</div>
          }
          <form className="login-box__form">
            <LabeledField
              type="text"
              label="Email"
              name="email"
              onKeyUp={this.saveInput.bind(this, 'email')}
            />

            <LabeledField
              type="password"
              label="Password"
              name="password"
              onKeyUp={this.saveInput.bind(this, 'password')}
            />

            <button type="submit"
                    className="login-box__form-submit"
                    onClick={this.submitForm.bind(this)}
            >
              Login
            </button>
          </form>
          <a href="https://someaddressforgetapass">
            Reset your password
          </a>
        </div>
      </div>
    );
  }
}

LoginBox.propTypes = {
  onSubmit: PropTypes.func.isRequired,
  error: PropTypes.string
};

export default LoginBox;


/// test
describe('LoginBox', () => {
  it('should render without throwing an error', () => {
    const wrapper = shallow(
      <LoginBox onSubmit={() => {}} error="testing"/>
    );
    expect(wrapper).toBe.ok;
  });

  it('should update state with keyUp events', () => {
    const wrapper = mount(
      <LoginBox onSubmit={() => {}}/>
    );
    const input = wrapper.find('input[type="text"]');
    input.simulate('keyUp', {target: {value: "a"}});
    expect(wrapper.state().email).toEqual('a');
  });

  it('should submit the form when enter is pressed', () => {
    const onSubmit = jest.fn();
    const wrapper = mount(
      <LoginBox onSubmit={onSubmit}/>
    );
    const input = wrapper.find('input[type="text"]');
    input.simulate('keyUp', {keyCode: 13});
    expect(onSubmit).toHaveBeenCalled();
  });
});

```

In this test `shallow` and `mount` from Enzyme are the stars of the show.  
Shallow calls constructor and render functions in the component.  
Mount calls constructor, render and componentDidMount functions in the component.

Enzyme also includes a lot of useful functions for finding elements and interacting  
with them through event simulation like click, keyUp, focus, etc.  
The state and workflow of a component can be tested with Enzyme utilities.  
With these tools we can ensure what events are triggered on each case, if there's an  
accidental change for example on the onSubmit action the test will fail and will alert  
that something not wanted has changed.