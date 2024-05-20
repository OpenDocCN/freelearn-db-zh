# 第五章：多文档 ACID 事务

MongoDB 在 2018 年 7 月发布的 4.0 版本中引入了多文档原子性、一致性、隔离性和持久性（ACID）事务。事务是关系数据库的一个组成部分。从早期开始，每个关系数据库管理系统（RDBMS）都依赖事务来实现 ACID。在非关系型数据库中实现这些功能是一个突破，可以从根本上改变开发人员和数据库架构师设计软件系统的方式。

在上一章中，我们学习了如何使用 Ruby、Python 和 PHP 的驱动程序和框架查询 MongoDB。在本章中，我们将学习以下主题：

+   多文档 ACID 事务

+   使用 Ruby 和 Python 进行事务处理

# 背景

MongoDB 是一个非关系型数据库，对 ACID 提供了很少的保证。MongoDB 中的数据建模并不是围绕 BCNF、2NF 和 3NF 规范化，而是相反的方向。

在 MongoDB 中，很多时候，最好的方法是将我们的数据嵌入子文档中，这样可以得到比在关系数据库管理系统中单行数据更自包含的文档。这意味着一个逻辑事务可以多次影响单个文档。在 MongoDB 中，单文档事务是符合 ACID 的，这意味着多文档 ACID 事务对于 MongoDB 的开发并不是必不可少的。

然而，有几个原因说明为什么实现多文档事务是一个好主意。多年来，MongoDB 已经从一个小众数据库发展成为一个多用途数据库，被各种公司广泛使用，从初创公司到财富 500 强公司。在许多不同的用例中，不可避免地会有一些情况，数据建模无法或不应该将数据放入子文档和数组中。此外，即使对于数据架构师来说，今天最好的解决方案是嵌入数据，他们也无法确定这种情况会一直持续下去。这使得选择正确的数据库层变得困难。

关系型数据库数据建模已经存在了 40 多年，是一个众所周知和理解的数据建模过程。帮助数据架构师以熟悉的方式工作总是一个额外的好处。

在引入多文档事务之前，唯一的解决方法是在应用层以定制的方式实现它们。这既耗时又容易出错。在应用层实现两阶段提交过程也可能更慢，并导致增加数据库锁。

在本章中，我们将专注于使用原生的 MongoDB 事务，因为这是 MongoDB 公司强烈推荐的。

# ACID

ACID 代表原子性、一致性、隔离性和持久性。在接下来的章节中，我们将解释这些对于我们的数据库设计和架构意味着什么。

# 原子性

原子性指的是事务需要是原子的概念。要么成功并且其结果对每个后续用户都是可见的，要么失败并且每个更改都被回滚到开始之前的状态。事务中的所有操作要么全部发生，要么全部不发生。

一个简单的例子来理解原子性是将钱从账户*A*转账到账户*B*。需要从账户*A*中存入钱，然后转入账户*B*。如果操作在中途失败，那么账户*A*和*B*都需要恢复到操作开始之前的状态。

在 MongoDB 中，即使操作跨多个子文档或数组，单个文档中的操作也总是原子的。

跨多个文档的操作需要使用 MongoDB 事务来实现原子性。

# 一致性

一致性指的是数据库始终处于一致的状态。每个数据库操作可能成功完成、失败或中止；然而，最终，我们的数据库必须处于一个数据一致的状态。

数据库约束必须始终得到尊重。任何未来的事务也必须能够查看过去事务更新的数据。在实践中，分布式数据系统中最常用的一致性模型是最终一致性。

最终一致性保证一旦我们停止更新数据，所有未来的读取将最终读取最新提交的写入值。在分布式系统中，这是性能方面唯一可接受的模型，因为数据需要在不同服务器之间的网络上复制。

相比之下，最不受欢迎的强一致性模型保证每次未来读取都将始终读取上次提交的写入值。这意味着每次更新都会在下一次读取之前传播并提交到每个服务器，这将对这些系统的性能造成巨大压力。

MongoDB 介于最终一致性和严格一致性之间。事实上，MongoDB 采用因果一致性模型。在因果一致性中，任何事务执行顺序都与如果所有因果相关的读/写操作按照反映它们因果关系的顺序执行的顺序相同。

在实践中，这意味着并发操作可能以不同的顺序被看到，读取对应于最新写入的值，与它们因果依赖的写入相关。

最终，这是一个权衡，即同时发生多少并发操作和应用程序读取的数据一致性。

# 隔离

隔离指的是事务操作对其他并行操作的可见性。

隔离级别至关重要的一个例子描述如下情景：

+   事务*A*将用户 1 的账户余额从 50 更新为 100，但不提交事务。

+   事务*B*将用户 1 的账户余额读取为 100。

+   事务*A*被回滚，将用户 1 的账户余额恢复为 50。

+   事务*B*认为用户 1 有 100 英镑，而实际上只有 50 英镑。

+   事务*B*更新用户 2 的值，增加 100 英镑。用户 2 从用户 1 那里凭空得到 100 英镑，因为用户 1 的账户中只有 50 英镑。我们的想象中的银行陷入了麻烦。

隔离通常有四个级别，从最严格到最不严格，如下所示：

+   可串行化

+   重复读取

+   读已提交

+   读未提交

我们可能遇到的问题，从最不严重到最严重，取决于隔离级别，如下所示：

+   幻读

+   不可重复读取

+   脏读

+   丢失更新

在任何数据库中，丢失有关操作更新的数据是最糟糕的事情，因为这将使我们的数据库无法使用，并使其成为一个不可信任的数据存储。这就是为什么在每个隔离级别中，即使是读取未提交的隔离，也不会丢失数据。

然而，其他三个问题也可能出现。我们将在以下部分简要解释这些问题是什么。

# 幻读

幻读发生在事务过程中，另一个事务通过添加或删除属于其结果集的行来修改其结果集。一个例子是：

+   事务*A*查询所有用户。返回 1,000 个用户，但事务没有提交。

+   事务*B*添加另一个用户；现在我们的数据库中有 1,001 个用户。

+   事务*A*第二次查询所有用户。现在返回 1,001 个用户。事务*A*现在提交。

在严格的可串行化隔离级别下，事务*B*应该被阻止添加新用户，直到事务*A*提交其事务。当然，这可能会在数据库中引起巨大的争用，并导致性能下降，因为每个更新操作都需要等待读取提交其事务。这就是为什么通常在实践中很少使用可串行化。

# 不可重复读取

当在事务期间检索一行两次并且行的值在每次读取操作时都不同时，就会发生不可重复读取。

根据先前的资金转移示例，我们可以类似地说明不可重复读取：

+   事务*B*读取用户 1 的账户余额为 50。

+   事务*A*将用户 1 的账户余额从 50 更新为 100，并提交事务。

+   事务*B*再次读取用户 1 的账户余额，并获得新值 100，然后提交事务。

问题在于事务*B*在其事务过程中得到了不同的值，因为它受到了事务*A*的更新的影响。这是一个问题，因为事务*B*在其自己的事务中得到了不同的值。然而，在实践中，它解决了在用户不存在时在用户之间转移资金的问题。

这就是为什么读取提交的隔离级别在实践中是最常用的隔离级别，它不会阻止不可重复读取，但会阻止脏读。

# 脏读

先前的例子中，我们通过虚拟货币赚了钱，最终从只有 50 英镑余额的账户中转出了 100 英镑，这是脏读的典型例子。

读取未提交的隔离级别不能保护我们免受脏读的影响，这就是为什么它在生产级系统中很少使用的原因。

以下是隔离级别与潜在问题的对比：

| **隔离级别** | **丢失更新** | **脏读** | **不可重复读** | **幻读** |
| --- | --- | --- | --- | --- |
| 读取未提交 | 不会发生 | 可能发生 | 可能发生 | 可能发生 |
| 读取提交 | 不会发生 | 不会发生 | 可能发生 | 可能发生 |
| 可重复读取 | 不会发生 | 不会发生 | 不会发生 | 可能发生 |
| 可串行化 | 不会发生 | 不会发生 | 不会发生 | 不会发生 |

PostgreSQL 使用默认（和可配置的）读取提交的隔离级别。由于 MongoDB 本质上不是关系数据库管理系统，对每个操作使用事务会使情况变得更加复杂。

在这些术语中，等价的隔离级别是读取未提交。根据先前给出的例子，这可能看起来很可怕，但另一方面，在 MongoDB 中，（再次强调，一般情况下）没有事务的概念或回滚事务。读取未提交指的是在使其持久化之前将更改可见。有关持久化部分的更多详细信息将在持久性的以下部分中提供。

# 耐久性

在关系数据库系统中，持久性是指每个成功提交的事务将在面对故障时幸存的属性。这通常指的是将已提交事务的内容写入持久存储（如硬盘或 SDD）。关系数据库管理系统总是通过将每个已提交的事务写入事务日志或**预写式日志**（**WAL**）来遵循持久性概念。MongoDB 使用 WiredTiger 存储引擎，每 60 毫秒将写入使用 WAL 提交到其基于持久存储的日志，并且在所有实际目的上是持久的。由于持久性很重要，每个数据库系统都更喜欢首先放松 ACID 的其他方面，而持久性通常是最后放松的。

# 我们何时需要在 MongoDB 中使用 ACID？

现有的原子性保证，对于单文档操作，MongoDB 可以满足大多数现实世界应用程序的完整性需求。然而，一些传统上受益于 ACID 事务的用例在 MongoDB 中建模可能比使用众所周知的 ACID 范式要困难得多。

毫不奇怪，许多这些情况来自金融行业。处理资金和严格的监管框架意味着每个操作都需要被存储，有时需要严格的执行顺序，记录，验证，并且在需要时可以进行审计。构建数字银行需要在 MongoDB 中将多个账户之间的交互表示为文档。

管理用户或执行高频交易的算法产生的大量财务交易，还需要验证每一笔交易。这些交易可能涉及多个文档，因为它们可能再次涉及多个帐户。

使用多文档 ACID 事务的一般模式是当我们可以拥有无限数量的实体时，有时可能达到数百万。在这种情况下，对实体进行子文档和数组建模是行不通的，因为文档最终会超出 MongoDB 中内置的 16 MB 文档大小限制。

# 使用 MongoDB 构建数字银行

多文档 ACID 事务的最常见用例来自金融领域。在本节中，我们将使用事务模拟数字银行，并逐渐介绍如何利用事务来获益的更复杂的示例。

银行必须提供的基本功能是帐户和在它们之间转移货币金额。在引入事务之前，MongoDB 开发人员有两个选择。第一种选择是 MongoDB 的处理方式，即将数据嵌入文档中，可以是子文档或值数组。对于帐户，这可能会导致以下代码块中的数据结构：

```sql
{accounts: [ {account_id: 1, account_name: ‘alex’, balance: 100}, {account_id: 2, account_name: ‘bob’, balance: 50}]}
```

然而，即使在这种简单的格式中，它也会很快超出 MongoDB 中固定的 16 MB 文档限制。这种方法的优势在于，由于我们必须处理单个文档，所有操作都将是原子的，从而在我们将资金从一个帐户转移到另一个帐户时产生强一致性保证。

除了使用关系数据库之外，唯一可行的替代方案是在应用程序级别实现保证，以模拟具有适当代码的事务，以便在出现错误时撤消部分或全部事务。这种方法可以奏效，但会导致更长的上市时间，并且更容易出现错误。

MongoDB 的多文档 ACID 事务方法类似于我们在关系数据库中处理事务的方式。从 MongoDB Inc.于 2018 年 6 月发布的《MongoDB 多文档 ACID 事务》白皮书中，最简单的例子是，MongoDB 中的通用事务将如下代码块所示：

```sql
s.start_transaction()
orders.insert_one(order, session=s)
stock.update_one(item, stockUpdate, session=s)
s.commit_transaction()
```

然而，在 MySQL 中进行相同的事务将如下所示：

```sql
db.start_transaction()
cursor.execute(orderInsert, orderData)
cursor.execute(stockUpdate, stockData)
db.commit()
```

也就是说，在现代 Web 应用程序框架中，大多数情况下，事务都隐藏在对象关系映射（ORM）层中，对应用程序开发人员不可见。框架确保 Web 请求被包装在传递到底层数据库层的事务中。这在 ODM 框架中还没有实现，但可以预期这种情况可能会发生改变。

# 设置我们的数据

我们将使用一个包含两个帐户的示例`init_data.json`文件。Alex 有 100 个 hypnotons 虚拟货币，而 Mary 有 50 个：

```sql
{"collection": "accounts", "account_id": "1", "account_name": "Alex", "account_balance":100}{"collection": "accounts", "account_id": "2", "account_name": "Mary", "account_balance":50}
```

使用以下 Python 代码，我们可以将这些值插入到我们的数据库中：

```sql
import json
class InitData:
   def __init__(self):
       self.client = MongoClient('localhost', 27017)
       self.db = self.client.mongo_bank
       self.accounts = self.db.accounts
       # drop data from accounts collection every time to start from a clean slate
       self.accounts.drop()
       # load data from json and insert them into our database
       init_data = InitData.load_data(self)
       self.insert_data(init_data)
   @staticmethod
   def load_data(self):
       ret = []
       with open('init_data.json', 'r') as f:
           for line in f:
               ret.append(json.loads(line))
       return ret
   def insert_data(self, data):
       for document in data:
           collection_name = document['collection']
           account_id = document['account_id']
           account_name = document['account_name']
           account_balance = document['account_balance']
           self.db[collection_name].insert_one({'account_id': account_id, 'name': account_name, 'balance': account_balance})
```

这将导致我们的`mongo_bank`数据库在我们的`accounts`集合中具有以下文档：

```sql
> db.accounts.find()
{ "_id" : ObjectId("5bc1fa7ef8d89f2209d4afac"), "account_id" : "1", "name" : "Alex", "balance" : 100 }
{ "_id" : ObjectId("5bc1fa7ef8d89f2209d4afad"), "account_id" : "2", "name" : "Mary", "balance" : 50 }
```

# 帐户之间的转移-第一部分

作为 MongoDB 开发人员，模拟事务的最常见方法是在代码中实现基本检查。对于我们的示例帐户文档，您可能会尝试实现帐户转移如下：

```sql
   def transfer(self, source_account, target_account, value):
       print(f'transferring {value} Hypnotons from {source_account} to {target_account}')
       with self.client.start_session() as ses:
           ses.start_transaction()
           self.accounts.update_one({'account_id': source_account}, {'$inc': {'balance': value*(-1)} })
           self.accounts.update_one({'account_id': target_account}, {'$inc': {'balance': value} })
           updated_source_balance = self.accounts.find_one({'account_id': source_account})['balance']
           updated_target_balance = self.accounts.find_one({'account_id': target_account})['balance']
           if updated_source_balance < 0 or updated_target_balance < 0:
               ses.abort_transaction()
           else:
               ses.commit_transaction()
```

在 Python 中调用此方法将从帐户 1 转移 300 个 hypnotons 到帐户 2：

```sql
>>> obj = InitData.new
>>> obj.transfer('1', '2', 300)
```

这将导致以下结果：

```sql
> db.accounts.find()
{ "_id" : ObjectId("5bc1fe25f8d89f2337ae40cf"), "account_id" : "1", "name" : "Alex", "balance" : -200 }
{ "_id" : ObjectId("5bc1fe26f8d89f2337ae40d0"), "account_id" : "2", "name" : "Mary", "balance" : 350 }
```

这里的问题不在于`updated_source_balance`和`updated_target_balance`的检查。这两个值都反映了分别为`-200`和`350`的新值。问题也不在于`abort_transaction()`操作。相反，问题在于我们没有使用会话。

在 MongoDB 中学习有关事务的最重要的一点是，我们需要使用会话对象来包装事务中的操作；但与此同时，在事务代码块内部仍然可以执行事务范围之外的操作。

这里发生的是，我们启动了一个事务会话，如下所示：

```sql
       with self.client.start_session() as ses:
```

然后完全忽略了这一点，通过非事务方式进行所有更新。然后我们调用了`abort_transaction`，如下所示：

```sql
               ses.abort_transaction()
```

要中止的事务本质上是无效的，没有任何需要回滚的内容。

# 账户之间的转账-第二部分

实现事务的正确方法是在每个我们想要在最后提交或回滚的操作中使用会话对象，如下所示：

```sql
   def tx_transfer_err(self, source_account, target_account, value):
       print(f'transferring {value} Hypnotons from {source_account} to {target_account}')
       with self.client.start_session() as ses:
           ses.start_transaction()
           res = self.accounts.update_one({'account_id': source_account}, {'$inc': {'balance': value*(-1)} }, session=ses)
           res2 = self.accounts.update_one({'account_id': target_account}, {'$inc': {'balance': value} }, session=ses)
           error_tx = self.__validate_transfer(source_account, target_account)

           if error_tx['status'] == True:
               print(f"cant transfer {value} Hypnotons from {source_account} ({error_tx['s_bal']}) to {target_account} ({error_tx['t_bal']})")
               ses.abort_transaction()
           else:
               ses.commit_transaction()
```

现在唯一的区别是我们在两个更新语句中都传递了`session=ses`。为了验证我们是否有足够的资金来实际进行转账，我们编写了一个辅助方法`__validate_transfer`，其参数是源和目标账户 ID：

```sql
   def __validate_transfer(self, source_account, target_account):
       source_balance = self.accounts.find_one({'account_id': source_account})['balance']
       target_balance = self.accounts.find_one({'account_id': target_account})['balance']

       if source_balance < 0 or target_balance < 0:
          return {'status': True, 's_bal': source_balance, 't_bal': target_balance}
       else:
           return {'status': False}
```

不幸的是，这次尝试也会*失败*。原因与之前相同。当我们在事务内部时，对数据库进行的更改遵循 ACID 原则。事务内部的更改对外部查询不可见，直到它们被提交。

# 账户之间的转账-第三部分

解决转账问题的正确实现将如下代码所示（完整的代码示例附在代码包中）：

```sql
from pymongo import MongoClient
import json

class InitData:
   def __init__(self):
       self.client = MongoClient('localhost', 27017, w='majority')
       self.db = self.client.mongo_bank
       self.accounts = self.db.accounts

       # drop data from accounts collection every time to start from a clean slate
       self.accounts.drop()

       init_data = InitData.load_data(self)
       self.insert_data(init_data)
       self.transfer('1', '2', 300)

   @staticmethod
   def load_data(self):
       ret = []
       with open('init_data.json', 'r') as f:
           for line in f:
               ret.append(json.loads(line))
       return ret

   def insert_data(self, data):
       for document in data:
           collection_name = document['collection']
           account_id = document['account_id']
           account_name = document['account_name']
           account_balance = document['account_balance']

           self.db[collection_name].insert_one({'account_id': account_id, 'name': account_name, 'balance': account_balance})

   # validating errors, using the tx session
   def tx_transfer_err_ses(self, source_account, target_account, value):
       print(f'transferring {value} Hypnotons from {source_account} to {target_account}')
       with self.client.start_session() as ses:
           ses.start_transaction()
           res = self.accounts.update_one({'account_id': source_account}, {'$inc': {'balance': value * (-1)}}, session=ses)
           res2 = self.accounts.update_one({'account_id': target_account}, {'$inc': {'balance': value}}, session=ses)
           error_tx = self.__validate_transfer_ses(source_account, target_account, ses)

           if error_tx['status'] == True:
               print(f"cant transfer {value} Hypnotons from {source_account} ({error_tx['s_bal']}) to {target_account} ({error_tx['t_bal']})")
               ses.abort_transaction()
           else:
               ses.commit_transaction()

   # we are passing the session value so that we can view the updated values
   def __validate_transfer_ses(self, source_account, target_account, ses):
       source_balance = self.accounts.find_one({'account_id': source_account}, session=ses)['balance']
       target_balance = self.accounts.find_one({'account_id': target_account}, session=ses)['balance']
       if source_balance < 0 or target_balance < 0:
           return {'status': True, 's_bal': source_balance, 't_bal': target_balance}
       else:
           return {'status': False}

def main():
   InitData()

if __name__ == '__main__':
   main()
```

在这种情况下，通过传递会话对象的`ses`值，我们确保可以使用`update_one()`在数据库中进行更改，并且还可以使用`find_one()`查看这些更改，然后执行`abort_transaction()`操作或`commit_transaction()`操作。

事务无法执行**数据定义语言**（**DDL**）操作，因此`drop()`，`create_collection()`和其他可能影响 MongoDB 的 DDL 的操作在事务内部将失败。这就是为什么我们在`MongoClient`对象中设置`w='majority'`，以确保当我们在开始事务之前删除集合时，此更改将对事务可见。

即使我们明确注意在事务期间不创建或删除集合，也有一些操作会隐式执行此操作。

我们需要确保在尝试插入或更新（更新和插入）文档之前集合存在。

最后，如果需要回滚，使用事务时我们不需要跟踪先前的账户余额值，因为 MongoDB 将放弃事务范围内所做的所有更改。

继续使用 Ruby 进行相同的示例，我们有第三部分的以下代码：

```sql
require 'mongo'

class MongoBank
  def initialize
    @client = Mongo::Client.new([ '127.0.0.1:27017' ], database: :mongo_bank)
    db = @client.database
    @collection = db[:accounts]

    # drop any existing data
    @collection.drop

    @collection.insert_one('collection': 'accounts', 'account_id': '1', 'account_name': 'Alex', 'account_balance':100)
    @collection.insert_one('collection': 'accounts', 'account_id': '2', 'account_name': 'Mary', 'account_balance':50)

    transfer('1', '2', 30)
    transfer('1', '2', 300)
  end

  def transfer(source_account, target_account, value)
    puts "transferring #{value} Hypnotons from #{source_account} to #{target_account}"
    session = @client.start_session

    session.start_transaction(read_concern: { level: :snapshot }, write_concern: { w: :majority })
    @collection.update_one({ account_id: source_account }, { '$inc' => { account_balance: value*(-1)} }, session: session)
    @collection.update_one({ account_id: target_account }, { '$inc' => { account_balance: value} }, session: session)

    source_account_balance = @collection.find({ account_id: source_account }, session: session).first['account_balance']

    if source_account_balance < 0
      session.abort_transaction
    else
      session.commit_transaction
    end
  end

end

# initialize class
MongoBank.new
```

除了 Python 示例中提出的所有观点之外，我们还发现可以根据事务自定义`read_concern`和`write_concern`。

多文档 ACID 事务的可用`read_concern`级别如下：

+   `majority`：复制集中大多数服务器已确认数据。为了使事务按预期工作，它们还必须使用`write_concern`为`majority`。

+   `local`：只有本地服务器已确认数据。

+   `snapshot`：截至 MongoDB 4.0，事务的默认`read_concern`级别。如果事务以`write_concern`为`majority`提交，所有事务操作将从大多数提交数据的快照中读取，否则无法做出保证。

事务的读关注点设置在事务级别或更高级别（会话或最终客户端）。不支持在单个操作中设置读关注点，并且通常不建议这样做。

多文档 ACID 事务的可用`write_concern`级别与 MongoDB 中的其他地方相同，除了不支持`w:0`（无确认）。

# 使用 MongoDB 的电子商务

对于我们的第二个示例，我们将在三个不同的集合上使用更复杂的事务用例。

我们将使用 MongoDB 模拟电子商务应用程序的购物车和付款交易过程。使用我们将在本节末尾提供的示例代码，我们将首先用以下数据填充数据库。

我们的第一个集合是`users`集合，每个用户一个文档：

```sql
> db.users.find()
{ "_id" : ObjectId("5bc22f35f8d89f2b9e01d0fd"), "user_id" : 1, "name" : "alex" }
{ "_id" : ObjectId("5bc22f35f8d89f2b9e01d0fe"), "user_id" : 2, "name" : "barbara" }
```

然后我们有一个`carts`集合，每个购物车一个文档，通过`user_id`与我们的用户相关联：

```sql
> db.carts.find()
{ "_id" : ObjectId("5bc2f9de8e72b42f77a20ac8"), "cart_id" : 1, "user_id" : 1 }
{ "_id" : ObjectId("5bc2f9de8e72b42f77a20ac9"), "cart_id" : 2, "user_id" : 2 }
```

`payments`集合保存通过的任何已完成付款，存储`cart_id`和`item_id`以链接到它所属的购物车和已支付的商品：

```sql
> db.payments.find()
{ "_id" : ObjectId("5bc2f9de8e72b42f77a20aca"), "cart_id" : 1, "name" : "alex", "item_id" : 101, "status" : "paid" }
```

最后，`inventories`集合保存我们当前可用的物品数量（按`item_id`），以及它们的价格和简短描述：

```sql
> db.inventories.find()
{ "_id" : ObjectId("5bc2f9de8e72b42f77a20acb"), "item_id" : 101, "description" : "bull bearing", "price" : 100, "quantity" : 5 }
```

在这个例子中，我们将演示使用 MongoDB 的模式验证功能。使用 JSON 模式，我们可以定义一组验证，每次插入或更新文档时都会在数据库级别进行检查。这是一个相当新的功能，因为它是在 MongoDB 3.6 中引入的。在我们的情况下，我们将使用它来确保我们的库存中始终有正数的物品数量。

MongoDB shell 格式中的`validator`对象如下：

```sql
validator = { validator:
 { $jsonSchema:
 { bsonType: "object",
 required: ["quantity"],
 properties:
 { quantity:
 { bsonType: ["long"],
 minimum: 0,
 description: "we can’t have a negative number of items in our inventory"
 }
 }
 }
 }
}
```

JSON 模式可用于实现我们在 Rails 或 Django 中通常在模型中具有的许多验证。我们可以定义这些关键字如下表所示：

| **关键字** | **验证类型** | **描述** |
| --- | --- | --- |
| `enum` | 所有 | 字段中允许的值的枚举。 |
| `type` | 所有 | 字段中允许的类型的枚举。 |
| `minimum`/`maximum` | 数值 | 数值字段的最小值和最大值。 |
| `minLength`/`maxLength` | 字符串 | 字符串字段允许的最小和最大长度。 |
| `pattern` | 字符串 | 字符串字段必须匹配的正则表达式模式。 |
| `required` | 对象 | 文档必须包含在 required 属性数组中定义的所有字符串。 |
| `minItems`/`maxItems` | 数组 | 数组中的项的最小和最大长度。 |
| `uniqueItems` | 数组 | 如果设置为 true，则数组中的所有项必须具有唯一值。 |
| `title` | N/A | 开发人员使用的描述性标题。  |
| `description` | N/A | 开发人员使用的描述。 |

使用 JSON 模式，我们可以将验证从我们的模型转移到数据库层和/或使用 MongoDB 验证作为 Web 应用程序验证的额外安全层。

要使用 JSON 模式，我们必须在创建集合时指定它，如下所示：

```sql
> db.createCollection("inventories", validator)
```

回到我们的例子，我们的代码将模拟拥有五个滚珠轴承的库存，并下两个订单；用户 Alex 订购两个滚珠轴承，然后用户 Barbara 订购另外四个滚珠轴承。

如预期的那样，第二个订单不会通过，因为我们的库存中没有足够的滚珠来满足它。我们将在以下代码中看到这一点：

```sql
from pymongo import MongoClient
from pymongo.errors import ConnectionFailure
from pymongo.errors import OperationFailure

class ECommerce:
   def __init__(self):
       self.client = MongoClient('localhost', 27017, w='majority')
       self.db = self.client.mongo_bank
       self.users = self.db['users']
       self.carts = self.db['carts']
       self.payments = self.db['payments']
       self.inventories = self.db['inventories']
       # delete any existing data
       self.db.drop_collection('carts')
       self.db.drop_collection('payments')
       self.db.inventories.remove()
       # insert new data
       self.insert_data()
       alex_order_cart_id = self.add_to_cart(1,101,2)
       barbara_order_cart_id = self.add_to_cart(2,101,4)
       self.place_order(alex_order_cart_id)
       self.place_order(barbara_order_cart_id)
   def insert_data(self):
       self.users.insert_one({'user_id': 1, 'name': 'alex' })
       self.users.insert_one({'user_id': 2, 'name': 'barbara'})
       self.carts.insert_one({'cart_id': 1, 'user_id': 1})
       self.db.carts.insert_one({'cart_id': 2, 'user_id': 2})
       self.db.payments.insert_one({'cart_id': 1, 'name': 'alex', 'item_id': 101, 'status': 'paid'})
       self.db.inventories.insert_one({'item_id': 101, 'description': 'bull bearing', 'price': 100, 'quantity': 5.0})

   def add_to_cart(self, user, item, quantity):
       # find cart for user
       cart_id = self.carts.find_one({'user_id':user})['cart_id']
       self.carts.update_one({'cart_id': cart_id}, {'$inc': {'quantity': quantity}, '$set': { 'item': item} })
       return cart_id

   def place_order(self, cart_id):
           while True:
               try:
                   with self.client.start_session() as ses:
                       ses.start_transaction()
                       cart = self.carts.find_one({'cart_id': cart_id}, session=ses)
                       item_id = cart['item']
                       quantity = cart['quantity']
                       # update payments
                       self.db.payments.insert_one({'cart_id': cart_id, 'item_id': item_id, 'status': 'paid'}, session=ses)
                       # remove item from cart
                       self.db.carts.update_one({'cart_id': cart_id}, {'$inc': {'quantity': quantity * (-1)}}, session=ses)
                       # update inventories
                       self.db.inventories.update_one({'item_id': item_id}, {'$inc': {'quantity': quantity*(-1)}}, session=ses)
                       ses.commit_transaction()
                       break
               except (ConnectionFailure, OperationFailure) as exc:
                   print("Transaction aborted. Caught exception during transaction.")
                   # If transient error, retry the whole transaction
                   if exc.has_error_label("TransientTransactionError"):
                       print("TransientTransactionError, retrying transaction ...")
                       continue
                   elif str(exc) == 'Document failed validation':
                       print("error validating document!")
                       raise
                   else:
                       print("Unknown error during commit ...")
                       raise
def main():
   ECommerce()
if __name__ == '__main__':
   main()
```

我们将把前面的例子分解为有趣的部分，如下所示：

```sql
   def add_to_cart(self, user, item, quantity):
       # find cart for user
       cart_id = self.carts.find_one({'user_id':user})['cart_id']
       self.carts.update_one({'cart_id': cart_id}, {'$inc': {'quantity': quantity}, '$set': { 'item': item} })
       return cart_id
```

`add_to_cart()`方法不使用事务。原因是因为我们一次只更新一个文档，这些操作是原子操作。

然后，在`place_order()`方法中，我们启动会话，然后随后在此会话中启动事务。与前一个用例类似，我们需要确保在我们想要在事务上下文中执行的每个操作的末尾添加`session=ses`参数：

```sql
    def place_order(self, cart_id):
 while True:
 try:
 with self.client.start_session() as ses:
 ses.start_transaction()
 …
 # update payments
 self.db.payments.insert_one({'cart_id': cart_id, 'item_id': item_id, 'status': 'paid'}, session=ses)
 # remove item from cart
 self.db.carts.update_one({'cart_id': cart_id}, {'$inc': {'quantity': quantity * (-1)}}, session=ses)
 # update inventories
 self.db.inventories.update_one({'item_id': item_id}, {'$inc': {'quantity': quantity*(-1)}}, session=ses)
 ses.commit_transaction()
 break
 except (ConnectionFailure, OperationFailure) as exc:
 print("Transaction aborted. Caught exception during transaction.")
 # If transient error, retry the whole transaction
 if exc.has_error_label("TransientTransactionError"):
 print("TransientTransactionError, retrying transaction ...")
 continue
 elif str(exc) == 'Document failed validation':
 print("error validating document!")
 raise
 else:
 print("Unknown error during commit ...")
 raise
```

在这种方法中，我们使用可重试事务模式。我们首先将事务上下文包装在`while True`块中，从本质上使其永远循环。然后我们在一个`try`块中包含我们的事务，它将监听异常。

`transient transaction`类型的异常，具有`TransientTransactionError`错误标签，将导致在`while True`块中继续执行，从而从头开始重试事务。另一方面，验证失败或任何其他错误将在记录后重新引发异常。

`session.commitTransaction()`和`session.abortTransaction()`操作将被 MongoDB 重试一次，无论我们是否重试事务。

在这个例子中，我们不需要显式调用`abortTransaction()`，因为 MongoDB 会在面对异常时中止它。

最后，我们的数据库看起来像下面的代码块：

```sql
> db.payments.find()
{ "_id" : ObjectId("5bc307178e72b431c0de385f"), "cart_id" : 1, "name" : "alex", "item_id" : 101, "status" : "paid" }
{ "_id" : ObjectId("5bc307178e72b431c0de3861"), "cart_id" : 1, "item_id" : 101, "status" : "paid" }
```

我们刚刚进行的付款没有名称字段，与我们在滚动事务之前插入数据库的示例付款相反：

```sql
> db.inventories.find()
{ "_id" : ObjectId("5bc303468e72b43118dda074"), "item_id" : 101, "description" : "bull bearing", "price" : 100, "quantity" : 3 }
```

我们的库存中有正确数量的滚珠轴承，三个（五减去 Alex 订购的两个），如下面的代码块所示：

```sql
> db.carts.find()
{ "_id" : ObjectId("5bc307178e72b431c0de385d"), "cart_id" : 1, "user_id" : 1, "item" : 101, "quantity" : 0 }
{ "_id" : ObjectId("5bc307178e72b431c0de385e"), "cart_id" : 2, "user_id" : 2, "item" : 101, "quantity" : 4 }
```

我们的购物车中有正确的数量。 Alex 的购物车（`cart_id=1`）没有物品，而 Barbara 的购物车（`cart_id=2`）仍然有四个，因为我们没有足够的滚珠轴承来满足她的订单。我们的支付集合中没有 Barbara 订单的条目，库存中仍然有三个滚珠轴承。

我们的数据库状态是一致的，并且通过在应用程序级别实现中止事务和对账数据逻辑来节省大量时间。

在 Ruby 中继续使用相同的示例，我们有以下代码块：

```sql
require 'mongo'

class ECommerce
 def initialize
   @client = Mongo::Client.new([ '127.0.0.1:27017' ], database: :mongo_bank)
   db = @client.database
   @users = db[:users]
   @carts = db[:carts]
   @payments = db[:payments]
   @inventories = db[:inventories]

   # drop any existing data
   @users.drop
   @carts.drop
   @payments.drop
   @inventories.delete_many

   # insert data
   @users.insert_one({ "user_id": 1, "name": "alex" })
   @users.insert_one({ "user_id": 2, "name": "barbara" })

   @carts.insert_one({ "cart_id": 1, "user_id": 1 })
   @carts.insert_one({ "cart_id": 2, "user_id": 2 })

   @payments.insert_one({"cart_id": 1, "name": "alex", "item_id": 101, "status": "paid" })
   @inventories.insert_one({"item_id": 101, "description": "bull bearing", "price": 100, "quantity": 5 })

   alex_order_cart_id = add_to_cart(1, 101, 2)
   barbara_order_cart_id = add_to_cart(2, 101, 4)

   place_order(alex_order_cart_id)
   place_order(barbara_order_cart_id)
 end

 def add_to_cart(user, item, quantity)
   session = @client.start_session
   session.start_transaction
   cart_id = @users.find({ "user_id": user}).first['user_id']
   @carts.update_one({"cart_id": cart_id}, {'$inc': { 'quantity': quantity }, '$set': { 'item': item } }, session: session)
   session.commit_transaction
   cart_id
 end

 def place_order(cart_id)
   session = @client.start_session
   session.start_transaction
   cart = @carts.find({'cart_id': cart_id}, session: session).first
   item_id = cart['item']
   quantity = cart['quantity']
   @payments.insert_one({'cart_id': cart_id, 'item_id': item_id, 'status': 'paid'}, session: session)
   @carts.update_one({'cart_id': cart_id}, {'$inc': {'quantity': quantity * (-1)}}, session: session)
   @inventories.update_one({'item_id': item_id}, {'$inc': {'quantity': quantity*(-1)}}, session: session)
   quantity = @inventories.find({'item_id': item_id}, session: session).first['quantity']
   if quantity < 0
     session.abort_transaction
   else
     session.commit_transaction
   end
 end
end

ECommerce.new
```

与 Python 代码示例类似，我们在每个操作中传递`session: session`参数，以确保我们在事务内进行操作。

在这里，我们没有使用可重试的事务模式。无论如何，MongoDB 都会在抛出异常之前重试提交或中止事务一次。

# 多文档 ACID 事务的最佳实践和限制

在开发中使用 MongoDB 4.0.3 版本的事务时，目前存在一些限制和最佳实践：

+   事务超时设置为 60 秒。

+   作为最佳实践，任何事务不应尝试修改超过 1,000 个文档。在事务期间读取文档没有限制。

+   oplog 将记录事务的单个条目，这意味着这受到 16MB 文档大小限制的影响。对于更新文档的事务来说，这并不是一个大问题，因为 oplog 只会记录增量。然而，当事务插入新文档时，这可能会成为一个问题，此时 oplog 将记录新文档的全部内容。

+   我们应该添加应用程序逻辑来处理失败的事务。这可能包括使用可重试写入，或者在错误无法重试或我们已经耗尽重试时执行一些业务逻辑驱动的操作（通常意味着自定义 500 错误）。

+   诸如修改索引、集合或数据库之类的 DDL 操作将排队等待活动事务。在 DDL 操作仍在进行时尝试访问命名空间的事务将立即中止。

+   事务只在副本集中起作用。从 MongoDB 4.2 开始，事务也将适用于分片集群。

+   节制使用；在开发中使用 MongoDB 事务时可能要考虑的最重要的一点是，它们并不是用来替代良好模式设计的。只有在没有其他方法可以对数据进行建模时才应该使用它们。

# 总结

在本章中，我们了解了在 MongoDB 的上下文中关于 ACID 的教科书关系数据库理论。

然后，我们专注于多文档 ACID 事务，并在 Ruby 和 Python 中应用它们到两个用例中。我们了解了何时使用 MongoDB 事务以及何时不使用它们，如何使用它们，它们的最佳实践和限制。

在下一章中，我们将处理 MongoDB 最常用的功能之一 - 聚合。
