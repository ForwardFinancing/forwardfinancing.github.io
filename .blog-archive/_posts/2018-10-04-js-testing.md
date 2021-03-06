---
layout: post
title:  "Javascript Unit Testing"
date:   2018-10-04 12:00:00 -0400
categories: software unit testing js javascript jest enzyme
author: Tania Paiva
---

Should I test this? the answer is a **big yes**. Tests are your friends, and saviors sometimes (often).

At Forward Financing, we feel proud when our projects reach high test coverage. It's our best insurance  
for making improvements and upgrades later with the guarantee that existing functionality will  
keep working.

Tests watched our backs during a recent upgrade process, saving us from potential headaches. When the  
test failed after a package upgrade we discovered an issue with the way the css classes were generated  
in one of our custom libraries. This issue was affecting the UI for five of our services.  

Mistakes can always happen, and that is where tests can be real saviors as an alert that  
something was altered by mistake or that some functionality is not providing the expected result.

Tests verify the application meets the requirements, it guarantees the code does what we expect  
it to do.

Okay, tests can do a lot for us, but it is not always sun and butterflies — writing good tests takes  
time and it gets better with some experience.

![Coverage]({{"/assets/test_coverage.png" | absolute_url}})

The following are some of the testing tools we use in our javascript projects.

* [Jest](https://jestjs.io/)
"Delightful JavaScript Testing" (according to them of course) is used by Facebook to test all JavaScript code  
including React applications. One of Jest's philosophies is to provide an integrated "zero-configuration" experience.

* [Enzyme](https://airbnb.io/enzyme/)
Is a JavaScript Testing utility for React that makes it easier to assert, manipulate, and traverse your  
React Components' output.

Ok, those are cool names but what can we do with them?  
Here are some examples on how to use these tools:

We have a utility function that validates dates are in the following format DD/MM/YYYY

``` javascript
function isValid(value){
  if (value.length > 0) {
    if (value.split("/").length == 3 &&
        value.split("/")[2].length == 4 &&
        value.split("/")[1].length == 2 &&
        value.split("/")[0].length == 2
        ) {
        return true;
    } else {
      return false;
    }
  } else {
    return false;
  }
}

  // the test
  describe('utils', function() {
    it('should have a function that validates a string date in format DD/MM/YYYY', function () {
      const invalidYear = "01/04/22";
      expect(utils.isValid(invalidYear)).toBe.false;

      const invalidMonth = "01/4/2000";
      expect(utils.isValid(invalidMonth)).toBe.false;

      const invalidDay = "1/02/1990";
      expect(utils.isValid(invalidDay)).toBe.false;

      const valueWithAllValid = "02/02/2000";
      expect(utils.isValid(valueWithAllValid)).toBe.true;
    });
  });
```

This first example makes use of Jest to set what we expect to see with each set of values.  
The behavior of the function on each case is evaluated and we can set the expected result  
according the params.

Now how does Jest and Enzyme work with React components? :thinking:  
Let's say we have a login box component with fields for email and password that sends the  
form when clicking the button or when hitting enter key.

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
Shallow calls `constructor` and `render` functions in the component.  
Mount calls `constructor`, `render` and `componentDidMount` functions in the component.

Enzyme also includes a lot of useful functions for finding elements and interacting  
with them through event simulation like click, keyUp, focus, etc.  
The state and workflow of a component can be tested with Enzyme utilities.  
With these tools we can ensure what events are triggered on each case, if there's an  
accidental change for example on the onSubmit action the test will fail and will alert  
that something not wanted has changed.

In general terms we can say we are fans of high test coverage because of the multiple benefits  
that provide to the development process and the maintenance of the code.