Backend Express Starter
---
An express backend starter to make it easier to do new projects. All the basic user auth and routes should be ready to go. The only thing that should need to be generated after cloning is all of the database things. Instructions are below ↓

Installation Instructions:
---
- Clone repo
- cd into repo directory
- `npm install`
- `touch .env` in root directory
- Copy format of `.envexample` for database information
- Create user for database (make sure to have username the same in `.env` file) -> `psql -c "CREATE USER <username> PASSWORD '<password>' CREATEDB"`
- Create a database -> `npx dotenv sequelize db:create`

Database Commands:
---
- Create models -> `npx sequelize model:generate --name User --attributes username:string,email:string,hashedPassword:string`
- Apply constraints to migration file ->
    ```
    'use strict';
    module.exports = {
    up: (queryInterface, Sequelize) => {
        return queryInterface.createTable('Users', {
        id: {
            allowNull: false,
            autoIncrement: true,
            primaryKey: true,
            type: Sequelize.INTEGER
        },
        username: {
            type: Sequelize.STRING(30),
            allowNull: false,
            unique: true,
        },
        email: {
            type: Sequelize.STRING(256),
            allowNull: false,
            unique: true,
        },
        hashedPassword: {
            type: Sequelize.STRING.BINARY,
            allowNull: false,
        },
        createdAt: {
            allowNull: false,
            type: Sequelize.DATE,
            defaultValue: Sequelize.fn('now'),
        },
        updatedAt: {
            allowNull: false,
            type: Sequelize.DATE,
            defaultValue: Sequelize.fn('now'),
        }
        });
    },
    down: (queryInterface, Sequelize) => {
        return queryInterface.dropTable('Users');
    }
    };
    ```
- Migrate -> `npx dotenv sequelize db:migrate`
- Apply constraints to model ->
    ```
    'use strict';
    const { Validator } = require('sequelize');

    module.exports = (sequelize, DataTypes) => {
    const User = sequelize.define('User', {
        username: {
        type: DataTypes.STRING,
        allowNull: false,
        validate: {
            len: [4, 30],
            isNotEmail(value) {
            if (Validator.isEmail(value)) {
                throw new Error('Cannot be an email.');
            }
            },
        },
        },
        email: {
        type: DataTypes.STRING,
        allowNull: false,
        validate: {
            len: [3, 256]
        },
        },
        hashedPassword: {
        type: DataTypes.STRING.BINARY,
        allowNull: false,
        validate: {
            len: [60, 60]
        },
        },
    }, {});
    User.associate = function(models) {
        User.hasMany(models.List, {
        foreignKey: 'userId'
        });
    };

    User.prototype.toSafeObject = function () {
        const { id, username, email } = this;
        return { id, username, email };
    };
    User.prototype.validatePassword = function (password) {
        return bcrypt.compareSync(password, this.hashedPassword.toString());
    };
    User.getCurrentUserById = async function (id) {
        return await User.scope('currentUser').findByPk(id);
    };
    User.login = async function ({ credential, password }) {
        const { Op } = require('sequelize');
        const user = await User.scope('loginUser').findOne({
        where: {
            [Op.or]: {
            username: credential,
            email: credential,
            },
        },
        });
        if (user && user.validatePassword(password)) {
        return await User.scope('currentUser').findByPk(user.id);
        }
    };
    User.signup = async function ({ username, email, password }) {
        const hashedPassword = bcrypt.hashSync(password);
        const user = await User.create({
        username,
        email,
        hashedPassword
        });
        return await User.scope('currentUser').findByPk(user.id);
    }
    return User;
    };
    ```
- Generate seed -> `npx sequelize seed:generate --name demo-user`
- Fill seed file with ->
    ```
    'use strict';
    const faker = require('faker');
    const bcrypt = require('bcryptjs');

    module.exports = {
    up: (queryInterface, Sequelize) => {
        return queryInterface.bulkInsert('Users', [
        {
            email: 'demo@user.io',
            username: 'Demo-lition',
            hashedPassword: bcrypt.hashSync('password'),
        },
        {
            email: faker.internet.email(),
            username: 'FakeUser1',
            hashedPassword: bcrypt.hashSync(faker.internet.password()),
        },
        {
            email: faker.internet.email(),
            username: 'FakeUser2',
            hashedPassword: bcrypt.hashSync(faker.internet.password()),
        },
        ], {});
    },

    down: (queryInterface, Sequelize) => {
        const Op = Sequelize.Op;
        return queryInterface.bulkDelete('Users', {
        username: { [Op.in]: ['Demo-lition', 'FakeUser1', 'FakeUser2'] }
        }, {});
    }
    };
    ```
- Seed database -> `npx dotenv sequelize db:seed:all`
