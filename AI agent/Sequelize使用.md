# Sequelize 使用笔记

Sequelize 是一个基于 Promise 的 Node.js ORM（对象关系映射）框架，支持 PostgreSQL、MySQL、MariaDB、SQLite 和 Microsoft SQL Server。它让你用 JavaScript 对象的方式操作数据库，无需编写原生 SQL。

## 📦 安装与配置

### 安装依赖

```bash
npm install sequelize mysql2   # MySQL 驱动
# 或根据不同数据库选择：pg pg-hstore（PostgreSQL）、sqlite3、tedious（SQL Server）
```

### 环境变量配置（推荐使用 dotenv）

创建 `.env` 文件：

```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASS=123456
DB_NAME=myapp
DB_DIALECT=mysql
DB_SYNC=false   # 是否自动同步表结构（开发时可设为 true）
```

### 基础配置与 Sequelize 实例

```javascript
// config/database.js
require('dotenv').config();
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASS,
  {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: process.env.DB_DIALECT || 'mysql',
    logging: process.env.NODE_ENV === 'production' ? false : console.log,
    pool: { max: 5, min: 0, acquire: 30000, idle: 10000 },
    define: {
      timestamps: true,      // 自动添加 createdAt, updatedAt
      underscored: true,      // 将驼峰转为下划线（如 createdAt -> created_at）
      charset: 'utf8mb4',
      collate: 'utf8mb4_unicode_ci'
    }
  }
);

module.exports = sequelize;
```

---

## 🏗️ 定义模型（Model）

模型对应数据库中的一张表。每个模型文件通常放在 `models/` 目录下。

### 基础模型定义

```javascript
// models/user.model.js
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const User = sequelize.define('User', {   // 模型名（单数），表名默认为复数 'Users'
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  username: {
    type: DataTypes.STRING(50),
    allowNull: false,
    unique: true,
    validate: {
      len: [3, 20]   // 长度校验
    }
  },
  email: {
    type: DataTypes.STRING(100),
    unique: true,
    validate: {
      isEmail: true
    }
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false
  },
  age: {
    type: DataTypes.INTEGER,
    validate: {
      min: 0,
      max: 120
    }
  },
  status: {
    type: DataTypes.TINYINT,
    defaultValue: 1
  }
}, {
  // 额外配置
  tableName: 'users',           // 自定义表名，否则为模型名的复数
  timestamps: true,             // 覆盖全局配置
  paranoid: true,               // 启用软删除（添加 deletedAt 字段）
  indexes: [{ unique: true, fields: ['email'] }]
});

module.exports = User;
```

### 数据类型（DataTypes）

常用类型：

| Sequelize 类型 | 描述 |
|----------------|------|
| `STRING(255)`  | 可变字符串 |
| `TEXT`         | 长文本 |
| `INTEGER`      | 整数 |
| `BIGINT`       | 大整数 |
| `FLOAT`        | 浮点数 |
| `DOUBLE`       | 双精度浮点数 |
| `DECIMAL(10,2)`| 定点数 |
| `BOOLEAN`      | 布尔值 |
| `DATE`         | 日期时间 |
| `DATEONLY`     | 仅日期 |
| `ENUM('a','b')`| 枚举 |
| `JSON`         | JSON 对象 |
| `UUID`         | UUID 字符串 |

---

## 🔄 模型同步

将模型定义的 schema 同步到数据库。

```javascript
// db/index.js
const sequelize = require('./config/database');
const models = require('./models');   // 集中导出所有模型

async function syncDatabase() {
  try {
    await sequelize.authenticate();   // 测试连接
    console.log('数据库连接成功');

    if (process.env.DB_SYNC === 'true') {
      // force: true 会删除表并重建（危险！），alter: true 尝试修改表结构
      await sequelize.sync({ alter: true });
      console.log('数据库同步完成');
    }
  } catch (err) {
    console.error('数据库连接失败:', err);
    process.exit(1);
  }
}

syncDatabase();

module.exports = { sequelize, models };
```

**注意**：生产环境不建议使用 `sync`，应通过迁移工具（migrations）管理表结构。

---

## ✨ CRUD 操作

假设我们已经有 `User` 模型。

### 创建（Create）

```javascript
// 方式一：build + save
const user = User.build({ username: 'alice', email: 'alice@example.com' });
await user.save();

// 方式二：create（直接保存）
const user = await User.create({
  username: 'bob',
  email: 'bob@example.com',
  password: '123456'
});

// 批量创建
const users = await User.bulkCreate([
  { username: 'u1', email: 'u1@e.com' },
  { username: 'u2', email: 'u2@e.com' }
]);
```

### 查询（Read）

```javascript
// 查询所有
const users = await User.findAll();

// 查询单个（根据主键）
const user = await User.findByPk(1);

// 查询单个（满足条件的第一条）
const user = await User.findOne({ where: { username: 'alice' } });

// 条件查询
const activeUsers = await User.findAll({
  where: { status: 1 },
  attributes: ['id', 'username'],  // 只返回指定字段
  order: [['createdAt', 'DESC']],
  limit: 10,
  offset: 0
});

// 使用操作符
const { Op } = require('sequelize');
const users = await User.findAll({
  where: {
    age: { [Op.gte]: 18 },          // age >= 18
    username: { [Op.like]: '%a%' }   // 模糊匹配
  }
});

// 计数
const count = await User.count({ where: { status: 1 } });
```

### 更新（Update）

```javascript
// 方式一：先查询再更新
const user = await User.findByPk(1);
user.username = 'new name';
await user.save();

// 方式二：直接更新（返回影响行数）
const [affectedCount] = await User.update(
  { email: 'new@example.com' },   // 更新字段
  { where: { id: 1 } }            // 条件
);

// 批量更新
await User.update(
  { status: 0 },
  { where: { age: { [Op.lt]: 18 } } }
);
```

### 删除（Delete）

```javascript
// 删除单个实例
const user = await User.findByPk(1);
await user.destroy();

// 条件删除（返回删除行数）
const deletedCount = await User.destroy({
  where: { status: 0 }
});

// 软删除（如果模型启用了 paranoid）
await User.destroy({ where: { id: 1 } });  // 实际上会更新 deletedAt 字段
// 恢复软删除的记录
await User.restore({ where: { id: 1 } });
```

---

## 🔗 关联关系

### 定义关联

在模型文件中定义，或统一在 `models/index.js` 中设置。

```javascript
// models/index.js
const User = require('./user.model');
const Post = require('./post.model');
const Profile = require('./profile.model');

// 一对多：一个用户有多个帖子
User.hasMany(Post, { foreignKey: 'userId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'userId', as: 'user' });

// 一对一：一个用户有一个个人资料
User.hasOne(Profile, { foreignKey: 'userId' });
Profile.belongsTo(User, { foreignKey: 'userId' });

// 多对多：学生与课程（需中间表 StudentCourse）
const Student = require('./student.model');
const Course = require('./course.model');
Student.belongsToMany(Course, { through: 'StudentCourses', foreignKey: 'studentId' });
Course.belongsToMany(Student, { through: 'StudentCourses', foreignKey: 'courseId' });

module.exports = { User, Post, Profile, Student, Course };
```

### 预加载（Eager Loading）

```javascript
// 查询用户并同时加载其帖子
const users = await User.findAll({
  include: {
    model: Post,
    as: 'posts',           // 与关联定义的 as 对应
    attributes: ['title'], // 只加载需要的字段
    where: { status: 'published' } // 可添加条件
  }
});

// 嵌套加载：用户 -> 帖子 -> 评论
const users = await User.findAll({
  include: {
    model: Post,
    include: ['comments']
  }
});

// 通过关联查询
const userWithPosts = await User.findByPk(1, {
  include: ['posts']
});
console.log(userWithPosts.posts); // 数组
```

### 关联创建与操作

```javascript
// 为已有用户创建帖子（自动设置 userId）
const user = await User.findByPk(1);
const post = await user.createPost({ title: '新帖子' });

// 获取关联数据
const posts = await user.getPosts();

// 移除关联（仅解除外键关系）
await user.removePost(postId);

// 添加关联（多对多）
await student.addCourse(courseId);
```

---

## 📝 查询进阶

### 常用操作符（Op）

| 操作符 | 含义 |
|--------|------|
| `[Op.eq]` | 等于 |
| `[Op.ne]` | 不等于 |
| `[Op.gt]` | 大于 |
| `[Op.lt]` | 小于 |
| `[Op.gte]` | 大于等于 |
| `[Op.lte]` | 小于等于 |
| `[Op.between]` | 介于之间 |
| `[Op.notBetween]` | 不在之间 |
| `[Op.in]` | 在列表中 |
| `[Op.notIn]` | 不在列表中 |
| `[Op.like]` | 模糊匹配 |
| `[Op.notLike]` | 不匹配 |
| `[Op.startsWith]` | 以...开头 |
| `[Op.endsWith]` | 以...结尾 |
| `[Op.substring]` | 包含子串 |
| `[Op.is]` | IS（常用于 IS NULL） |
| `[Op.not]` | NOT |

示例：

```javascript
const { Op } = require('sequelize');

const posts = await Post.findAll({
  where: {
    createdAt: {
      [Op.gte]: new Date('2025-01-01')
    },
    title: {
      [Op.like]: '%Sequelize%'
    }
  }
});
```

### 复杂条件（AND / OR）

```javascript
const users = await User.findAll({
  where: {
    [Op.or]: [
      { age: { [Op.lt]: 20 } },
      { status: 0 }
    ],
    [Op.and]: [
      { username: { [Op.ne]: 'admin' } },
      { createdAt: { [Op.gte]: '2025-01-01' } }
    ]
  }
});
```

### 排序、分组、限制

```javascript
const posts = await Post.findAll({
  where: { status: 'published' },
  order: [
    ['createdAt', 'DESC'],
    ['title', 'ASC']
  ],
  group: ['authorId'],
  having: { count: { [Op.gt]: 5 } },  // 与 group 搭配
  limit: 20,
  offset: 40
});
```

---

## 🔒 事务（Transaction）

### 托管事务（自动提交/回滚）

```javascript
const { sequelize } = require('./db');

try {
  const result = await sequelize.transaction(async (t) => {
    const user = await User.create({ username: 'new' }, { transaction: t });
    const profile = await Profile.create({ userId: user.id, bio: '...' }, { transaction: t });
    return { user, profile };
  });
  console.log('事务成功', result);
} catch (err) {
  console.error('事务回滚', err);
}
```

### 非托管事务（手动控制）

```javascript
const t = await sequelize.transaction();
try {
  const user = await User.create({ username: 'new' }, { transaction: t });
  await Profile.create({ userId: user.id, bio: '...' }, { transaction: t });
  await t.commit();
} catch (err) {
  await t.rollback();
  throw err;
}
```

---

## ⚙️ 钩子（Hooks）

生命周期事件，在特定操作前后自动执行。

```javascript
// 模型定义中添加钩子
User.beforeCreate((user, options) => {
  // 创建前的处理，如密码哈希
  user.password = bcrypt.hashSync(user.password, 10);
});

User.afterCreate((user, options) => {
  console.log(`新用户创建: ${user.username}`);
});

// 或在 define 时设置
const User = sequelize.define('User', { ... }, {
  hooks: {
    beforeCreate: (user) => { /* ... */ },
    afterUpdate: (user) => { /* ... */ }
  }
});
```

常用钩子：`beforeValidate`, `afterValidate`, `beforeCreate`, `afterCreate`, `beforeUpdate`, `afterUpdate`, `beforeDestroy`, `afterDestroy`, `beforeBulkCreate`, `afterBulkCreate` 等。

---

## 📋 常用方法速查表

| 操作 | 方法 |
|------|------|
| 查询所有 | `Model.findAll()` |
| 根据主键查询 | `Model.findByPk(id)` |
| 查询一条 | `Model.findOne({ where: ... })` |
| 创建 | `Model.create(data)` 或 `build + save` |
| 批量创建 | `Model.bulkCreate([...])` |
| 更新 | `Model.update(data, { where })` |
| 删除 | `Model.destroy({ where })` |
| 计数 | `Model.count({ where })` |
| 最大/最小值 | `Model.max('field')` / `Model.min('field')` |
| 求和 | `Model.sum('field', { where })` |
| 查找或创建 | `Model.findOrCreate({ where, defaults })` |
| 预加载 | `Model.findAll({ include: ... })` |
| 事务 | `sequelize.transaction(...)` |
| 原始 SQL 查询 | `sequelize.query('SELECT * FROM users')` |

---

## 🐛 调试与日志

- 在配置 Sequelize 时设置 `logging: console.log` 可打印 SQL 语句。
- 生产环境可关闭日志或重定向到 logger 文件。
- 使用 `sequelize.options.logging` 可以动态控制。

---

## 📚 学习资源

- [Sequelize 官方文档](https://sequelize.org/)
- [Sequelize v6 API 参考](https://sequelize.org/api/v6/)

---

这份笔记涵盖了 Sequelize 的核心用法，适合初学者快速上手和日常查阅。随着使用深入，可以进一步探索迁移（migrations）、模型作用域（scopes）、多态关联等高级特性。Happy coding!
