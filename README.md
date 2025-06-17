# Testing Node.js with Mocha, Chai and Sinon
 ![](https://komarev.com/ghpvc/?username=mscbuild) 
  ![](https://img.shields.io/github/license/mscbuild/e-learning) 
 ![][(https://img.shields.io/github/languages/code-size/mscbuild/testing_node-js)
![](https://img.shields.io/badge/PRs-Welcome-green)
![](https://img.shields.io/badge/code%20style-json-green)
![](https://img.shields.io/github/stars/mscbuild)
![](https://img.shields.io/badge/Topic-Github-lighred)
![](https://img.shields.io/website?url=https%3A%2F%2Fgithub.com%2Fmscbuild)

Tests help document the core functionality of an application. Well-written tests ensure that new features do not introduce changes that could break the application.

The engineer maintaining the codebase does not necessarily have to have written the source code. If the code is well tested, another engineer can confidently add new code or modify existing code and expect that these changes will not break other features or at least not cause unwanted side effects.

JavaScript and Node.js have many testing and assertion libraries, such as Jest, Jasmine, Qunit, and Mocha. In this article, we will look at how to use Mocha for testing, Chai for assertions, and Sinon for mocks, spies, and stubs.

# What is Unit Testing?

Unit tests verify that functions work as expected while being isolated from other components of the application. Unit tests allow you to test different functions in the application. There are several reasons to write them:

- Unit tests ensure that the code works as expected under different conditions.

- Unit tests help find errors in the code early in the development process.

- Since any tests that fail reveal defective code, writing unit tests helps build trust. You can be confident that your code is functional if all tests pass.

 # Project Setup

Let's create a new directory for our user application project:

~~~bash
mkdir mocha-unit-test && cd mocha-unit-test
mkdir src
~~~
Create a package.json file in the src folder and add the following code to it:
~~~bash
// src/package.json
{
  "name": "mocha-unit-test",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "mocha './src/**/*.test.js'",
    "start": "node src/app.js"
  },
  "keywords": [
    "mocha",
    "chai"
  ],
  "author": "Godwin Ekuma",
  "license": "ISC",
   "dependencies": {
    "dotenv": "^6.2.0",
    "express": "^4.18.2",
    "jsonwebtoken": "^8.5.1",
    "morgan": "^1.10.0",
    "mysql2": "^2.3.3",
    "pg": "^7.18.2",
    "pg-hstore": "^2.3.4",
    "sequelize": "^5.22.5"
  },
  "devDependencies": {
    "chai": "^4.3.7",
    "faker": "^4.1.0",
    "mocha": "^10.2.0",
    "sinon": "^15.0.1"
  }
}
~~~
Run `npm install to install the project dependencies.

Note that testing packages such as `mocha`, `chai, `sinon`, and `faker` are stored under dev-dependencies.

The `test` script uses a custom search pattern (glob) `./src/**/*.test.js` to set the path to test files. Mocha will look for test files (files ending in `.test.js`) in directories and subdirectories of the `src` folder.

# Repositories, services and controllers

- We will structure our application using the controller-service-repository pattern so that our application is divided into repositories, services, and controllers. This pattern separates the business layer of the application into three separate layers:

- The repository class is responsible for retrieving and writing data from the repository. The repository is used between the service layer and the model layer. For example, in UserRepository, you will create methods to write and read user data to and from the database.

- The service class calls the repository class and can combine their data to create new, more complex business objects. It is an abstraction between the controller and the repository. For example, UserService will be responsible for performing the logic needed to create a new user.

- The controller contains a minimal amount of logic and is used to call services. The controller rarely calls repositories directly unless there is a good reason to do so. The controller will perform basic validations on the data received from the services in order to send a response back to the client.

Splitting the application in this way makes testing easier.

# UserRepository class

Let's start by creating a repository class:
~~~bash
// src/user/user.repository.js
const { UserModel } = require("../database");
class UserRepository {
  constructor() {
    this.user = UserModel;
    this.user.sync({ force: true });
  }
  async create(name, email) {
    return this.user.create({
      name,
      email
    });
  }
  async getUser(id) {
    return this.user.findOne({ id });
  }
}
module.exports = UserRepository;
~~~
The `UserRepository` class has two methods: `create` and `getUser`. The `create` method adds a new user to the database, and the `getUser` method looks up a user in the database.

Let's test the `userRepository` methods below:
~~~bash
// src/user/user.repository.test.js
const chai = require("chai");
const sinon = require("sinon");
const expect = chai.expect;
const faker = require("faker");
const { UserModel } = require("../database");
const UserRepository = require("./user.repository");
describe("UserRepository", function() {
  const stubValue = {
    id: faker.random.uuid(),
    name: faker.name.findName(),
    email: faker.internet.email(),
    createdAt: faker.date.past(),
    updatedAt: faker.date.past()
  };
  describe("create", function() {
    it("should add a new user to the db", async function() {
      const stub = sinon.stub(UserModel, "create").returns(stubValue);
      const userRepository = new UserRepository();
      const user = await userRepository.create(stubValue.name, stubValue.email);
      expect(stub.calledOnce).to.be.true;
      expect(user.id).to.equal(stubValue.id);
      expect(user.name).to.equal(stubValue.name);
      expect(user.email).to.equal(stubValue.email);
      expect(user.createdAt).to.equal(stubValue.createdAt);
      expect(user.updatedAt).to.equal(stubValue.updatedAt);
    });
  });
});
~~~
In the code above, we test the create method of the `UserRepository` class. Note that we use a stub for the `UserModel`.create method. The stub is necessary because our goal is to test the repository, not the model. The faker library is used for test data.
~~~bash
// src/user/user.repository.test.js

const chai = require("chai");
const sinon = require("sinon");
const expect = chai.expect;
const faker = require("faker");
const { UserModel } = require("../database");
const UserRepository = require("./user.repository");

describe("UserRepository", function() {
  const stubValue = {
    id: faker.random.uuid(),
    name: faker.name.findName(),
    email: faker.internet.email(),
    createdAt: faker.date.past(),
    updatedAt: faker.date.past()
  };
   describe("getUser", function() {
    it("should retrieve a user with specific id", async function() {
      const stub = sinon.stub(UserModel, "findOne").returns(stubValue);
      const userRepository = new UserRepository();
      const user = await userRepository.getUser(stubValue.id);
      expect(stub.calledOnce).to.be.true;
      expect(user.id).to.equal(stubValue.id);
      expect(user.name).to.equal(stubValue.name);
      expect(user.email).to.equal(stubValue.email);
      expect(user.createdAt).to.equal(stubValue.createdAt);
      expect(user.updatedAt).to.equal(stubValue.updatedAt);
    });
  });
});
~~~
To test the `getUser` method, we also need to stub the `UserModel.findOne` method. We use `expect(stub.calledOnce).to.be.true` to assert that the stub was called at least once. The remaining assertions check that the value returned by the `getUser` method is correct.

***UserService class***
~~~bash
// src/user/user.service.js

const UserRepository = require("./user.repository");
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }
  async create(name, email) {
    return this.userRepository.create(name, email);
  }
  getUser(id) {
    return this.userRepository.getUser(id);
  }
}
module.exports = UserService;
~~~
The `UserService` class also has two methods: `create` and `getUser`. The `create` method calls the repository's `create` method, passing the name and email of the new user as arguments. The getUser method calls the repository's `getUser` method.

Let's test the `userService` methods below:
~~~bash
// src/user/user.service.test.js

const chai = require("chai");
const sinon = require("sinon");
const UserRepository = require("./user.repository");
const expect = chai.expect;
const faker = require("faker");
const UserService = require("./user.service");
describe("UserService", function() {
  describe("create", function() {
    it("should create a new user", async function() {
      const stubValue = {
        id: faker.random.uuid(),
        name: faker.name.findName(),
        email: faker.internet.email(),
        createdAt: faker.date.past(),
        updatedAt: faker.date.past()
      };
      const userRepo = new UserRepository();
      const stub = sinon.stub(userRepo, "create").returns(stubValue);
      const userService = new UserService(userRepo);
      const user = await userService.create(stubValue.name, stubValue.email);
      expect(stub.calledOnce).to.be.true;
      expect(user.id).to.equal(stubValue.id);
      expect(user.name).to.equal(stubValue.name);
      expect(user.email).to.equal(stubValue.email);
      expect(user.createdAt).to.equal(stubValue.createdAt);
      expect(user.updatedAt).to.equal(stubValue.updatedAt);
    });

//Тестирование случая, когда пользователь отсутствует.
    it("should return an empty object if no user matches the provided id", async function() {
      const stubValue = {};
      const userRepo = new UserRepository();
      const stub = sinon.stub(userRepo, "getUser").returns(stubValue);
      const userService = new UserService(userRepo);
      const user = await userService.getUser(1);
      expect(stub.calledOnce).to.be.true;
      expect(user).to.deep.equal({})
    });
  });
});
~~~
In the code above, we test the `create` method of the `UserService` service. We `create` a stub for the create method of the repository. The code below tests the `getUser` method of the service:
~~~bash
const chai = require("chai");
const sinon = require("sinon");
const UserRepository = require("./user.repository");
const expect = chai.expect;
const faker = require("faker");
const UserService = require("./user.service");
describe("UserService", function() {
  describe("getUser", function() {
    it("should return a user that matches the provided id", async function() {
      const stubValue = {
        id: faker.random.uuid(),
        name: faker.name.findName(),
        email: faker.internet.email(),
        createdAt: faker.date.past(),
        updatedAt: faker.date.past()
      };
      const userRepo = new UserRepository();
      const stub = sinon.stub(userRepo, "getUser").returns(stubValue);
      const userService = new UserService(userRepo);
      const user = await userService.getUser(stubValue.id);
      expect(stub.calledOnce).to.be.true;
      expect(user.id).to.equal(stubValue.id);
      expect(user.name).to.equal(stubValue.name);
      expect(user.email).to.equal(stubValue.email);
      expect(user.createdAt).to.equal(stubValue.createdAt);
      expect(user.updatedAt).to.equal(stubValue.updatedAt);
    });
  });
});
~~~
Here we again use the stub for the `getUser` method of the `UserRepository`. We also check that the stub was called at least once and that the returned value is correct.

***UserController class***
~~~bash
/ src/user/user.controller.js

class UserController {
  constructor(userService) {
    this.userService = userService;
  }
  async register(req, res, next) {
    const { name, email } = req.body;
    if (
      !name ||
      typeof name !== "string" ||
      (!email || typeof email !== "string")
    ) {
      return res.status(400).json({
        message: "Invalid Params"
      });
    }
    const user = await this.userService.create(name, email);
    return res.status(201).json({
      data: user
    });
  }
  async getUser(req, res) {
    const { id } = req.params;
    const user = await this.userService.getUser(id);
    return res.json({
      data: user
    });
  }
}
module.exports = UserController;
~~~
The `UserController` class has two methods: register and `getUser`. Each of these methods takes two parameters: the `req` and `res` objects.
~~~bash
// src/user/user.controller.test.js

describe("UserController", function() {
  describe("register", function() {
    let status json, res, userController, userService;
    beforeEach(() => {
      status = sinon.stub();
      json = sinon.spy();
      res = { json, status };
      status.returns(res);
      const userRepo = sinon.spy();
      userService = new UserService(userRepo);
    });
    it("should not register a user when name param is not provided", async function() {
      const req = { body: { email: faker.internet.email() } };
      await new UserController().register(req, res);
      expect(status.calledOnce).to.be.true;
      expect(status.args\[0\][0]).to.equal(400);
      expect(json.calledOnce).to.be.true;
      expect(json.args\[0\][0].message).to.equal("Invalid Params");
    });
    it("should not register a user when name and email params are not provided", async function() {
      const req = { body: {} };
      await new UserController().register(req, res);
      expect(status.calledOnce).to.be.true;
      expect(status.args\[0\][0]).to.equal(400);
      expect(json.calledOnce).to.be.true;
      expect(json.args\[0\][0].message).to.equal("Invalid Params");
    });
    it("should not register a user when email param is not provided", async function() {
      const req = { body: { name: faker.name.findName() } };
      await new UserController().register(req, res);
      expect(status.calledOnce).to.be.true;
      expect(status.args\[0\][0]).to.equal(400);
      expect(json.calledOnce).to.be.true;
      expect(json.args\[0\][0].message).to.equal("Invalid Params");
    });
    it("should register a user when email and name params are provided", async function() {
      const req = {
        body: { name: faker.name.findName(), email: faker.internet.email() }
      };
      const stubValue = {
        id: faker.random.uuid(),
        name: faker.name.findName(),
        email: faker.internet.email(),
        createdAt: faker.date.past(),
        updatedAt: faker.date.past()
      };
      const stub = sinon.stub(userService, "create").returns(stubValue);
      userController = new UserController(userService);
      await userController.register(req, res);
      expect(stub.calledOnce).to.be.true;
      expect(status.calledOnce).to.be.true;
      expect(status.args\[0\][0]).to.equal(201);
      expect(json.calledOnce).to.be.true;
      expect(json.args\[0\][0].data).to.equal(stubValue);
    });
  });
});
~~~
In the first three `it` blocks, we test that the user will not be created if one or both of the required parameters (email and name) are not passed. Note that we use a stub for `res.status` and set a spy on `res.json`:
~~~bash
describe("UserController", function() {
  describe("getUser", function() {
    let req;
    let res;
    let userService;
    beforeEach(() => {
      req = { params: { id: faker.random.uuid() } };
      res = { json: function() {} };
      const userRepo = sinon.spy();
      userService = new UserService(userRepo);
    });
    it("should return a user that matches the id param", async function() {
      const stubValue = {
        id: req.params.id,
        name: faker.name.findName(),
        email: faker.internet.email(),
        createdAt: faker.date.past(),
        updatedAt: faker.date.past()
      };
      const mock = sinon.mock(res);
      mock
        .expects("json")
        .once()
        .withExactArgs({ data: stubValue });
      const stub = sinon.stub(userService, "getUser").returns(stubValue);
      userController = new UserController(userService);
      const user = await userController.getUser(req, res);
      expect(stub.calledOnce).to.be.true;
      mock.verify();
    });
  });
});
~~~
To test the `getUser` method, we mocked the `json` method. Note that we also had to spy on the `UserRepository` by creating a new instance of `UserService`.

# Conclusion

Run the tests using the command below:
~~~bash
npm test
~~~
You should see that the tests have passed successfully.

> [!NOTE]
> So, we've looked at how you can use a combination of Mocha, Chai, and Sinon to create a robust test for your Node application.

> [!IMPORTANT]
> Be sure to check out their documentation to expand your knowledge of these tools

> [!TIP]
> The full code for this article can be found on [CodeSandbox](https://codesandbox.io/s/github/GodwinEkuma/mocha-chai-unit-test).

### 📝 License

This project is licensed under the `MIT License` - see the LICENSE file for details.

