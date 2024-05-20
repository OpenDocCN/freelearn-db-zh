# 附录

# 1. MongoDB 介绍

## 活动 1.01：设置电影数据库

**解决方案：**

以下步骤将帮助您完成此活动：

1.  首先，连接到作为*Exercise 1.04*的一部分设置的 MongoDB 集群，*在 Atlas 上设置您的第一个免费 MongoDB 集群*。它应该看起来像这样：

```js
mongo "mongodb+srv://cluster0-zlury.mongodb.net/test" –username   <yourUsername>
```

1.  在命令提示符上输入前面的命令，并在提示时提供密码。成功登录后，您应该看到带有您的集群名称的 shell 提示，类似于这样：

```js
MongoDB Enterprise Cluster0-shard-0:PRIMARY>
```

1.  现在，创建电影数据库并将其命名为`moviesDB`。利用`use`命令：

```js
use moviesDB
```

1.  创建一个带有几个相关属性的`movies`集合。通过将文档插入不存在的集合来创建集合。鼓励您考虑并实现您认为最合适的集合和属性：

```js
db.movies.insertMany(
    [
        {
            "title": "Rocky",
            "releaseDate": new Date("Dec 3, 1976"),
            "genre": "Action",
            "about": "A small-time boxer gets a supremely rare               chance to fight a heavy-weight champion in a bout                 in which he strives to go the distance for his                   self-respect.",
            "countries": ["USA"],
            "cast" : ["Sylvester Stallone","Talia Shire",              "Burt Young"],
            "writers" : ["Sylvester Stallone"],
            "directors" : ["John G. Avildsen"]
        },
        {
            "title": "Rambo 4",
            "releaseDate ": new Date("Jan 25, 2008"),
            "genre": "Action",
            "about": "In Thailand, John Rambo joins a group of               mercenaries to venture into war-torn Burma, and rescue                 a group of Christian aid workers who were kidnapped                   by the ruthless local infantry unit.",
            "countries": ["USA"],
            "cast" : [" Sylvester Stallone", "Julie Benz", "Matthew               Marsden"],
            "writers" : ["Art Monterastelli","Sylvester Stallone"],
            "directors" : ["Sylvester Stallone"]
        }
    ]
)
```

这应该导致以下输出：

```js
{
  "acknowledged" : true,
  "insertedIds" : [
    ObjectId("5f33d027592962df72246aed"),
    ObjectId("5f33d027592962df72246aee")
  ]
}
```

1.  使用`find`命令获取您在上一步中插入的文档，即`db.movies.find().pretty()`。它应该返回以下输出：

```js
{
      "_id" : ObjectId("5f33d027592962df72246aed"),
      "title" : "Rocky",
      "releaseDate" : ISODate("1976-12-02T13:00:00Z"),
      "genre" : "Action",
      "about" : "A small-time boxer gets a supremely rare chance to         fight a heavy-weight champion in a bout in which he strives to           go the distance for his self-respect.",
      "countries" : [
            "USA"
      ],
      "cast" : [
            "Sylvester Stallone",
            "Talia Shire",
            "Burt Young"
      ],
      "writers" : [
            "Sylvester Stallone"
      ],
      "directors" : [
            "John G. Avildsen"
      ]
}
{
      "_id" : ObjectId("5f33d027592962df72246aee"),
      "title" : "Rambo 4",
      "releaseDate " : ISODate("2008-01-24T13:00:00Z"),
      "genre" : "Action",
      "about" : "In Thailand, John Rambo joins a group of mercenaries         to venture into war-torn Burma, and rescue a group of           Christian aid workers who were kidnapped by the ruthless             local infantry unit.",
      "countries" : [
            "USA"
      ],
      "cast" : [
            " Sylvester Stallone",
            "Julie Benz",
            "Matthew Marsden"
      ],
      "writers" : [
            "Art Monterastelli",
            "Sylvester Stallone"
      ],
      "directors" : [
            "Sylvester Stallone"
      ]
}
{
      "_id" : ObjectId("5f33d050592962df72246aef"),
      "title" : "Rocky",
      "releaseDate" : ISODate("1976-12-02T13:00:00Z"),
      "genre" : "Action",
      "about" : "A small-time boxer gets a supremely rare chance to         fight a heavy-weight champion in a bout in which he strives to           go the
          distance for his self-respect.",
      "countries" : [
            "USA"
      ],
      "cast" : [
            "Sylvester Stallone",
            "Talia Shire",
            "Burt Young"
      ],
      "writers" : [
            "Sylvester Stallone"
      ],
      "directors" : [
            "John G. Avildsen"
      ]
}
{
      "_id" : ObjectId("5f33d050592962df72246af0"),
      "title" : "Rambo 4",
      "releaseDate " : ISODate("2008-01-24T13:00:00Z"),
      "genre" : "Action",
      "about" : "In Thailand, John Rambo joins a group of mercenaries         to venture into war-torn Burma, and rescue a group of           Christian aid
          workers who were kidnapped by the ruthless local             infantry unit.",
      "countries" : [
            "USA"
      ],
      "cast" : [
            " Sylvester Stallone",
            "Julie Benz",
            "Matthew Marsden"
      ],
      "writers" : [
            "Art Monterastelli",
            "Sylvester Stallone"
      ],
      "directors" : [
            "Sylvester Stallone"
      ]
}
```

1.  您可能还想在您的电影数据库中存储奖项信息。创建一个带有几条记录的`awards`集合。鼓励您考虑并提出自己的集合名称和属性。以下是在`awards`集合中插入几个示例文档的命令：

```js
db.awards.insertOne(
    {
        "title": "Oscars",
        "year": "1976",
        "category": "Best Film",
        "nominees": ["Rocky","All The President's Men","Bound For           Glory","Network","Taxi Driver"],
        "winners" :
        [
            {
                "movie" : "Rocky"
            }
        ]
    }
)
db.awards.insertOne(
    {
        "title": "Oscars",
        "year": "1976",
        "category": "Actor In A Leading Role",
        "nominees": ["PETER FINCH","ROBERT DE NIRO","GIANCARLO           GIANNINI"," WILLIAM HOLDEN","SYLVESTER STALLONE"],
        "winners" :
        [
            {
                "actor" : "PETER FINCH",
                "movie" : "Network"
            }
        ]
    }
)
```

这些命令中的每个都应该生成以下类似的输出：

```js
{
      "acknowledged" : true,
      "insertedId" : ObjectId("5f33d08e592962df72246af1")
}
```

这些命令中的每个都应该生成以下类似的输出：

```js
{
  "acknowledged" : true,
  "insertedId" : ObjectId("5f33d08e592962df72246af1")
}
```

注意

插入的 ID 是插入的文档的唯一 ID，因此它与前面的输出中提到的不同。

1.  运行`find`命令以从`awards`集合获取文档。以`//`（双斜杠）开头的行是注释，仅用于描述目的；数据库不会将其作为命令执行：

```js
// find all the documents from the awards collection
db.awards.find().pretty()
```

以下是前述命令的输出：

![图 1.39：来自奖项集合的文档](img/B15507_01_39.jpg)

图 1.39：来自奖项集合的文档

注意

这个练习是让您添加尽可能多的集合/文档，以便有效和高效地存储电影数据。随时添加任何其他相关的集合和文档。

在这个活动中，您为电影数据库找到了一个相关的数据库解决方案。您还在 MongoDB Atlas 上创建了一个用于存储集合和文档的数据库。

在下一章中，您将获得有关电影的另一个样本数据集的导入步骤。建议您实际考虑电影数据库所需的其他集合或集合中的属性。您还将在下一章中看到您的数据集与提供的样本有何不同。

# 2. 文档和数据类型

## 活动 2.01：将推文建模为 JSON 文档

**解决方案：**

执行以下步骤以完成活动：

1.  识别并列出可以包含在 JSON 文档中的推文中的以下字段：

```js
creation date and time
user id
user name 
user profile pic
user verification status
hash tags
mentions
tweet text
likes
comments
retweets
```

1.  将相关字段分组，以便它们可以作为嵌入对象或数组放置。由于推文可以有多个标签和提及，因此可以表示为数组。修改后的列表如下所示：

```js
creation date and time
user 
  id
  name 
  profile pic
  verification status
hash tags
  [tags]
mentions
  [mentions]
tweet text
likes
comments
retweets
```

1.  准备用户对象并从推文中添加值：

```js
{
  "id": "Lord_Of_Winterfell",
  "name": "Office of Ned Stark",
  "profile_pic": "https://user.profile.pic",
  "isVerified": true
}
```

1.  将所有标签列为数组：

```js
[
  "north",
  "WinterfellCares",
  "flueshots"
]
```

1.  将所有提及包括为数组：

```js
[
  "MaesterLuwin",
  "TheNedStark",
  "CatelynTheCat"
]
```

一旦将所有文档与其余字段组合，最终输出将如下所示：

```js
{
  "id": 1,
  "created_at": "Sun Apr 17 16:29:24 +0000 2011",
  "user": {
    "id": "Lord_Of_Winterfell",
    "name": "Office of Ned Stark",
    "profile_pic": "https://user.profile.pic",
    "isVerified": true
  },
  "text": "Tweeps in the #north. The long nights are upon us. Do            stock enough warm clothes, meat and mead…",
  "hashtags": [
    "north",
    "WinterfellCares",
    "flueshots"
  ],
  "mentions": [
    "MaesterLuwin",
    "TheNedStark",
    "CatelynTheCat"
  ],
  "likes_count": 14925,
  "retweet_count": 12165,
  "comments_count": 0
}
```

1.  单击`验证 JSON`以验证来自任何文本编辑器的代码：![图 2.21：经过验证的 JSON 文档](img/B15507_02_21.jpg)

图 2.21：经过验证的 JSON 文档

在这个活动中，您将数据从推文模型化为有效的 JSON 文档。

# 3. 服务器和客户端

## 活动 3.01：管理您的数据库用户

**解决方案：**

以下是该活动的详细步骤：

1.  转到 http://cloud.mongodb.com 连接到 Atlas 控制台。

1.  使用您的用户名和密码登录到新的 MongoDB Atlas 网络界面，该用户名和密码是在注册 Atlas Cloud 时创建的：![图 3.40：MongoDB Atlas 登录页面](img/B15507_03_40.jpg)

图 3.40：MongoDB Atlas 登录页面

1.  创建一个名为`dev_mflix`的新数据库，在 Atlas 集群页面上，点击`COLLECTIONS`按钮：![图 3.41：MongoDB Atlas 集群页面](img/B15507_03_41.jpg)

图 3.41：MongoDB Atlas 集群页面

一个包含所有集合的窗口将出现，如*图 3.42*所示：

![图 3.42：MongoDB Atlas 数据浏览器](img/B15507_03_42.jpg)

图 3.42：MongoDB Atlas 数据浏览器

1.  接下来，点击数据库列表顶部的`+Create Database`按钮。将出现以下窗口：![图 3.43：MongoDB 创建数据库窗口](img/B15507_03_43.jpg)

图 3.43：MongoDB 创建数据库窗口

1.  将`DATABASE NAME`设置为`dev_mflix`，将`COLLECTION NAME`设置为`dev_data01`，然后点击`CREATE`按钮。

1.  创建一个名为`Developers`的自定义角色。点击左侧的`Database Access`。在`Database Access`页面上，点击`Custom Role`选项卡。

1.  点击`Add Custom Role`按钮。`Add Custom Role`窗口将出现，如下截图所示：![图 3.44：Add Custom Role 窗口](img/B15507_03_44.jpg)

图 3.44：Add Custom Role 窗口

1.  在新的`Developers`角色中，在`dev_mflix`数据库上添加`readWrite`角色。然后，在`sample_mflix`数据库上添加`read`角色，然后点击`Add Custom Role`按钮。新的`Developers`角色将出现在列表中：![图 3.45：数据库访问-自定义角色](img/B15507_03_45.jpg)

图 3.45：数据库访问-自定义角色

1.  创建新的 Atlas 用户`Mark`。在`Database Access`菜单中，点击`+Add New Database User`按钮。`Add New Database User`窗口将如下图所示出现：![图 3.46：添加名为 Mark 的新用户](img/B15507_03_46.jpg)

图 3.46：添加名为 Mark 的新用户

1.  填写如下细节：

用户名：`Mark`

认证方法：`SCRAM`

预定义的自定义角色：`Developers`

现在，在 Atlas 用户列表中应该出现一个名为`Mark`的新用户：

![图 3.47：Atlas 数据库用户](img/B15507_03_47.jpg)

图 3.47：Atlas 数据库用户

1.  以`Mark`用户身份连接到 MongoDB 云数据库，并运行`db.getUser()` shell 函数。预期的 shell 输出如下截图所示：![图 3.48：Shell 输出（示例）](img/B15507_03_48.jpg)

图 3.48：Shell 输出（示例）

这就结束了这个活动。一个名为 Mark 的新开发人员已经被添加到 Atlas 系统，并且已经被授予适当的访问权限。

# 4. 查询文档

## 活动 4.01：按类型查找电影并分页显示结果

**解决方案：**

`findMoviesByGenre`函数最重要的部分是底层的 MongoDB 查询。您将采用逐步方法来解决问题，首先在 mongo shell 上创建查询。一旦查询准备好，您将把它封装到一个函数中：

1.  创建一个按`genre`过滤结果的查询。在这个活动中，我们使用`Action`类型的`genre`：

```js
      db.movies.find(
          {"genres" : "Action"}
      )
```

1.  要求仅返回电影的标题。为此，添加一个投影，仅投影`title`字段并排除其余部分，包括`_id`：

```js
      db.movies.find(
          {"genres" : "Action"},
          {"_id" : 0, "title" :1}
      )
```

1.  现在，按照 IMDb 评分的降序对结果进行排序。在查询中添加一个`sort()`函数：

```js
      db.movies.find(
          {"genres" : "Action"},
          {"_id" : 0, "title" :1})
      .sort({"imdb.rating" : -1})
```

1.  添加`skip`函数，现在提供任何您想要的值（在本例中为`3`）：

```js
      db.movies.find(
          {"genres" : "Action"}, 
          {"_id" : 0, "title" :1})
     .sort({"imdb.rating" : -1})
     .skip(3)
```

1.  接下来，在查询中添加一个`limit`，如下所示。限制值表示页面大小：

```js
      db.movies.find(
          {"genres" : "Action"}, 
          {"_id" : 0, "title" :1})
     .sort({"imdb.rating" : -1})
     .skip(3)
     .limit(5)
```

1.  最后，通过使用`toArray()`函数将结果游标转换为数组：

```js
     db.movies.find(
         {"genres" : "Action"}, 
         {"_id" : 0, "title" :1})
     .sort({"imdb.rating" : -1})
     .skip(3)
     .limit(5)
     .toArray()
```

1.  现在查询已经编写完成，打开文本编辑器并编写一个空函数，接受`genre`、页码和页大小，如下所示：

```js
      var findMoviesByGenre = function(genre, pageNumber, pageSize){
      }
```

1.  复制并粘贴查询到函数中，并将其赋值给一个变量，如下所示：

```js
      var findMoviesByGenre = function(genre, pageNumber, pageSize){
          var movies = db.movies.find(
              {"genres" : "Action"}, 
              {"_id" : 0, "title" :1})
          .sort({"imdb.rating" : -1})
          .skip(3)
          .limit(5)
          .toArray()
      }
```

1.  您将得到的结果是一个数组。编写需要迭代元素并打印`title`字段的逻辑，如下所示：

```js
      var findMoviesByGenre = function(genre, pageNumber, pageSize){
          var movies = db.movies.find(
              {"genres" : "Action"}, 
              {"_id" : 0, "title" :1})
          .sort({"imdb.rating" : -1})
          .skip(3)
          .limit(5)
          .toArray()
          print("************* Page : " + pageNumber)
          for(var i =0; i < movies.length; i++){
              print(movies[i].title)
          }
      }
```

1.  查询仍然具有硬编码的值，需要用作函数参数接收的变量替换这些值，因此将`genre`和`pageSize`变量放在正确的位置：

```js
      var findMoviesByGenre = function(genre, pageNumber, pageSize){

          var movies = db.movies.find(
              {"genres" : genre}, 
              {"_id" : 0, "title" :1})
          .sort({"imdb.rating" : -1})
          .skip(3)
          .limit(pageSize)
          .toArray()
          print("************* Page : " + pageNumber)
          for(var i =0; i < movies.length; i++){
              print(movies[i].title)
          }
      }
```

1.  现在，您需要根据页码和页面大小来确定跳过值。当用户在第一页时，跳过值应为零。在第二页时，跳过值应为页面大小。同样，如果用户在第三页，跳过值应为页面大小乘以 2。将此逻辑写成如下：

```js
      var findMoviesByGenre = function(genre, pageNumber, pageSize){
          var toSkip = 0;
          if(pageNumber < 2){
              toSkip = 0;
          } else{
              toSkip = (pageNumber -1) * pageSize;
          }
          var movies = db.movies.find(
              {"genres" : genre}, 
              {"_id" : 0, "title" :1})
          .sort({"imdb.rating" : -1})
          .skip(toSkip)
          .limit(pageSize)
          .toArray()
          print("************* Page : " + pageNumber)
          for(var i =0; i < movies.length; i++){
              print(movies[i].title)
          }
}
```

现在，在 limit 函数中使用新计算的 skip 值。这使函数完成。

1.  复制并粘贴该函数到 mongo shell 中并执行。您应该看到以下结果：

![图 4.46：最终输出](img/B15507_04_46.jpg)

图 4.46：最终输出

通过使用`sort()`、`skip()`和`limit()`函数，在此活动中，您为电影服务实现了分页，大大改善了用户体验。

# 5. 插入、更新和删除文档

## 活动 5.01：更新电影评论

解决方案：

执行以下步骤完成活动：

1.  首先，更新所有三条评论中的`movie_id`字段。由于我们需要对所有三条评论应用相同的更新，因此我们将使用`findOneAndUpdate()`函数以及`$set`运算符来更改字段的值：

```js
db.comments.updateMany(
  {
    "_id" : {$in : [
      ObjectId("5a9427658b0beebeb6975eb3"),
      ObjectId("5a9427658b0beebeb6975eb4"),
      ObjectId("5a9427658b0beebeb6975eaa")
    ]}
  },
  {
    $set : {"movie_id" : ObjectId("573a13abf29313caabd25582")}
  }
)
```

使用更新命令，我们通过其`_id`找到了三部电影，使用`$in`运算符提供它们的主键。然后，我们使用`$set`来更新字段`movie_id`的值。

1.  连接到 MongoDB Atlas 集群，使用数据库`sample_mflix`，然后执行上一步中的命令。输出应该如下：![图 5.30：将正确的电影分配给评论](img/B15507_05_30.jpg)

图 5.30：将正确的电影分配给评论

输出确认所有三条评论都已正确更新。

1.  通过`_id`找到电影`Sherlock Holmes`，并将评论数量减少`3`：

```js
db.movies.findOneAndUpdate(
  {"_id" : ObjectId("573a13bcf29313caabd57db6")},
  {$inc : {"num_mflix_comments" : -3}},
  {
    "returnNewDocument" : true,
    "projection" : {"title" : 1, "num_mflix_comments" : 1}
  }
)
```

此更新命令通过`_id`找到电影并使用负数的`$inc`来减少`num_mflix_comments`计数 3。它返回包含`title`和`num_mflix_comments`字段的修改后的文档。

1.  在同一个 mongo shell 上执行以下命令：![图 5.31：增加 Sherlock Holmes 的评论计数](img/B15507_05_31.jpg)

图 5.31：增加 Sherlock Holmes 的评论计数

输出显示评论数量正确减少了`3`。

1.  最后，在`50 First Dates`上准备一个类似的命令，并将评论数量增加`3`。应使用以下命令：

```js
db.movies.findOneAndUpdate(
  {"_id" : ObjectId("573a13abf29313caabd25582")},
  {$inc : {"num_mflix_comments" : 3}},
  {
    "returnNewDocument" : true,
    "projection" : {"title" : 1, "num_mflix_comments" : 1}
  }
)
```

在此更新操作中，我们通过`_id`找到电影，并使用`$inc`以正值 3 增加评论数量。它还返回更新后的文档，并仅返回`title`和`num_mflix_comments`字段。

1.  现在，在 mongo shell 上执行以下命令：![图 5.32：减少 50 First Dates 的评论计数](img/B15507_05_32.jpg)

图 5.32：减少 50 First Dates 的评论计数

输出显示评论数量已经正确增加。在此活动中，我们练习了修改不同集合的字段，并在更新操作期间增加和减少数值字段的值。

# 6. 使用聚合管道和数组进行更新

## 活动 6.01：将演员姓名添加到演员表中

**解决方案：**

执行以下步骤完成活动：

1.  由于只有一个电影文档需要更新，因此使用`findOneAndUpdate()`命令。打开文本编辑器并输入以下命令：

```js
      db.movies.findOneAndUpdate({"title" : "Jurassic World"})
```

此查询使用基于电影标题的查询表达式。

1.  准备一个更新表达式来将元素插入数组中。由于演员表必须是唯一的，因此使用`$addToSet`，如下所示：

```js
    db.movies.findOneAndUpdate(
        {"title" : "Jurassic World"},
        {$addToSet : {"cast" : "Nick Robinson"}}
    )
```

此查询将`Nick Robinson`插入`cast`，并确保不插入重复项。

1.  接下来，您需要对数组进行排序。由于集合是无序的，您不能在`$addToSet`表达式中使用`$sort`。相反，首先将元素添加到集合，然后对其进行排序。打开 mongo shell 并连接到`sample_mflix`数据库：

```js
    db.movies.findOneAndUpdate(
        {"title" : "Jurassic World"},
        {$addToSet : {"cast" : "Nick Robinson"}},
        {
            "returnNewDocument" : true,
            "projection" : {"_id" : 0, "title" : 1, "cast" : 1}
        }
    )
```

在此命令中，`returnNewDocument`标志已设置为`true`，并且只投影了`title`和`cast`字段。在`sample_mflix`数据库中执行查询：

![图 6.23：添加缺失的演员名字](img/B15507_06_23.jpg)

图 6.23：添加缺失的演员名字

截图确认元素`Nick Robinson`已正确添加到数组的末尾。

1.  打开文本编辑器，编写基本的更新命令，以及相同的查询表达式：

```js
db.movies.findOneAndUpdate(
    {"title" : "Jurassic World"}
)
```

1.  修改命令，向数组添加`$push`表达式，并提供`$sort`选项：

```js
    db.movies.findOneAndUpdate(
        {"title" : "Jurassic World"},
        {$push : {
            "cast" : {
                $each : [],
                $sort : 1
            }}
        }
    )
```

由于不需要推送新元素，因此已将空数组传递给`$each`运算符。

1.  添加`returnNewDocument`标志，将投影添加到`title`和`cast`字段，并执行命令，如下所示：

```js
    db.movies.findOneAndUpdate(
        {"title" : "Jurassic World"},
        {$push : {
            "cast" : {
                $each : [],
                $sort : 1
            }}
        },
        {
            "returnNewDocument" : true,
            "projection" : {"_id" : 0, "title" : 1, "cast" : 1}
        }
    )
```

1.  打开 mongo shell，连接到`sample_mflix`数据库，并执行命令：![图 6.24：对缺失的演员进行排序](img/B15507_06_24.jpg)

图 6.24：对缺失的演员进行排序

输出确认`cast`数组现在按元素的升序字母顺序排序。

# 7\. 数据聚合

## 活动 7.01：将聚合实践

**解决方案：**

执行以下步骤完成活动：

1.  首先，创建脚手架代码：

```js
// Chapter_7_Activity.js
var chapter7Activity = function() {
    var pipeline = [];
    db.movies.aggregate(pipeline).forEach(printjson);
}
Chapter7Activity()
```

1.  添加对 2001 年之前文档的第一个匹配项：

```js
var pipeline = [
    {$match: {
        released: {$lte: new ISODate("2001-01-01T00:00:00Z")}
    }}
  ];
```

1.  为至少获得一项奖项的电影添加第二个匹配条件：

```js
    {$match: {
        released: {$lte: new ISODate("2001-01-01T00:00:00Z")},
        "awards.wins": {$gte: 1},
    }}
```

1.  为奖项提名添加一个`sort`条件。这是为了确保我们`$group`语句中的`$first`运算符为每个类型获取了最高提名的电影：

```js
        {$sort: {
            "awards.nominations": -1
        }},
```

1.  添加`$group`阶段。根据第一个类型创建组，并输出每个组中的`$first`电影，以及该类型的奖项获奖总数：

```js
        { $group: {
            _id: {"$arrayElemAt": ["$genres", 0]},
            "film_id": {$first: "$_id"},
            "film_title": {$first: "$title"},
            "film_awards": {$first: "$awards"},
            "film_runtime": {$first: "$runtime"},
            "genre_award_wins": {$sum: "$awards.wins"},
          }},
```

在`comments`集合上执行连接，以检索每个组中电影的评论。这将我们计算的`film_id`字段与`movie_id`评论字段进行连接。将这个新数组命名为`comments`：

```js
          { $lookup: {
            from: "comments",
            localField: "film_id",
            foreignField: "movie_id",
            as: "comments"
        }},
```

1.  仅从新数组中投影第一个评论，以及您想要在最后输出的任何字段。使用`$slice`运算符仅返回`comments`数组中的第一个条目。还记得将预告片添加到电影的运行时间中：

```js
        { $project: {
            film_id: 1,
            film_title: 1,
            film_awards: 1,
            film_runtime: { $add: [ "$film_runtime", 12]},
            genre_award_wins: 1,
              "comments": { $slice: ["$comments", 1]}
          }}, 
```

1.  最后，按`genre_award_wins`排序，并限制为三个文档：

```js
          { $sort: {
              "genre_award_wins": -1}},
          { $limit: 3}
```

您的最终管道现在应该是这样的：

```js
var chapter7Activity = function() {
    var pipeline = [
        {$match: {
            released: {$lte: new ISODate("2001-01-01T00:00:00Z")},
            "awards.wins": {$gte: 1},
        }},
        {$sort: {
            "awards.nominations": -1}},
        { $group: {
            _id: {"$arrayElemAt": ["$genres", 0]},
            "film_id": {$first: "$_id"},
            "film_title": {$first: "$title"},
            "film_awards": {$first: "$awards"},
            "film_runtime": {$first: "$runtime"},
            "genre_award_wins": {$sum: "$awards.wins"},
          }},
          { $lookup: {
            from: "comments",
            localField: "film_id",
            foreignField: "movie_id",
            as: "comments"}},
        { $project: {
            film_id: 1,
            film_title: 1,
            film_awards: 1,
            film_runtime: { $add: [ "$film_runtime", 12]},
            genre_award_wins: 1,
              "comments": { $slice: ["$comments", 1]}
          }}, 
          { $sort: {
              "genre_award_wins": -1
          }},
          { $limit: 3}
    ];
    db.movies.aggregate(pipeline).forEach(printjson);
}
Chapter7Activity();
```

您的输出将如下所示：

![图 7.24：运行管道后的最终输出（为简洁起见截断）](img/B15507_07_24.jpg)

图 7.24：运行管道后的最终输出（为简洁起见截断）

在这个活动中，我们将聚合管道的不同方面组合在一起，以查询、转换和连接集合中的数据。通过结合本章学到的方法，您现在将能够自信地设计和编写高效的聚合管道，解决复杂的业务问题。

# 8\. 在 MongoDB 中编写 JavaScript 代码

## 活动 8.01：创建一个简单的 Node.js 应用程序

**解决方案：**

执行以下步骤完成活动：

1.  导入`readline`和 MongoDB 库：

```js
const readline = require('readline');
const MongoClient = require('mongodb').MongoClient;
```

1.  创建您的`readline`接口：

```js
const interface = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
});
```

1.  声明您需要的任何变量：

```js
const url = 'mongodb+srv://mike:password@myAtlas-  fawxo.gcp.mongodb.net/test?retryWrites=true&w=majority';
const client = new MongoClient(url);
const databaseName = "sample_mflix";
const collectionName = "movies";
```

1.  创建一个名为`list`的函数，该函数将获取给定类型的前五部电影，返回它们的`title`、`favourite`和`ID`字段。*您需要在此函数中请求类别。查看第 7.05 练习中的* `login` *方法，了解更多信息。将此与我们之前练习中的* `find` *代码结合起来*：

```js
const list = function(database, client) {
    interface.question("Please enter a category: ", (category) => {
        database.collection(collectionName).find({genres: { $all: [category]          }}).limit(5).project({title: 1, favourite:             1}).toArray(function(err, docs) {
            if(err) {
                console.log('Error in query.');
                console.log(err);
            }
            else if(docs) {
                console.log('Docs Array');
                console.log(docs);
            } else {
            }
            prompt(database, client);
            return;
         });
      });
}
```

1.  创建一个名为`favourite`的函数，它将通过标题更新一个文档，并向文档添加一个名为`favourite`的键，值为`true`。在这个函数中，你需要使用与`list`函数相同的方法来询问标题。将这个与我们之前练习的更新代码结合起来：

```js
const favourite = function(database, client) {
    interface.question("Please enter a movie title: ", (newTitle) => {
        database.collection(collectionName).updateOne({title: newTitle},           {$set: {favourite: true}}, function(err, result) {
            if(err) {
                console.log('Error updating');
                console.log(err);
                return false;
            }
            console.log('Updated documents #:');
            console.log(result.modifiedCount);
            prompt(database, client);
        })
    })
}
```

1.  基于用户输入创建一个交互式的`while`循环。如果你不确定如何做到这一点，请参考*Exercise 8.05*中的`prompt`函数，*在 Node.js 中处理输入*：

```js
const prompt = function(database, client) {
    interface.question("list, favourite OR exit: ", (input) => {
        if(input === "exit") {
            client.close();
            return interface.close(); // Will kill the loop.
        }   
        else if(input === "list") {
            list(database, client);
        }
        else if(input === "favourite") {
            favourite(database, client);
        }
        else { // If input matches none of our options.
            prompt(database, client)
        }
      });
}
```

1.  创建 MongoDB 连接和数据库，如果数据库创建成功，则调用你的`prompt`函数：

```js
client.connect(function(err) {
    if(err) {
        console.log('Failed to connect.');
        console.log(err);
        return false;
    }
    // Within the connection block, add a console.log to confirm the       connection
    console.log('Connected to MongoDB with NodeJS!');
    const database = client.db(databaseName);
    if(!database) {
        console.log('Database object doesn't exist!');
        return false;
    } else {
        prompt(database, client);
    }
})
```

记住，你需要通过每个函数传递`database`和`client`对象，包括每次调用`prompt`函数时。

1.  使用`node Activity8.01.js`运行你的代码。![图 8.9：最终输出（为简洁起见进行了截断）](img/B15507_08_09.jpg)

图 8.9：最终输出（为简洁起见进行了截断）

在这个活动中，你创建了一个带有交互式输入循环的应用程序，并实现了错误处理来处理用户输入的无效类型。

# 9\. 性能

## 活动 9.01：优化查询

**解决方案：**

执行以下步骤完成活动：

1.  打开你的 mongo shell 并连接到 Atlas 集群上的`sample_supplies`数据库。首先，你需要找出查询返回了多少条记录。以下片段显示了一个`count`查询，给出了在 Denver 店销售的背包数量：

```js
     db.sales.count(
         {
             "items.name" : "backpack",
             "storeLocation" : "Denver"
         }
     )

```

1.  查询返回了`711`条记录。

1.  接下来，使用`explain()`函数分析分析团队给出的查询，并打印执行统计数据，如下所示：

```js
db.sales.find(
         {
             "items.name" : "backpack",
             "storeLocation" : "Denver"
         },
         {
             "_id" : 0,
             "customer.email": 1,
             "customer.age": 1
         }
     ).sort({
         "customer.age" : -1
     }).explain("executionStats")
```

通过传递`executionStats`作为参数，查询调用了`explain()`函数。以下片段显示了输出的`executionStats`部分：

```js
"executionStats" : {
    "executionSuccess" : true,
    "nReturned" : 711,
    "executionTimeMillis" : 10,
    "totalKeysExamined" : 0,
    "totalDocsExamined" : 5000,
    executionStages" : {
        "stage" : "PROJECTION_DEFAULT",
        "nReturned" : 711,
        "executionTimeMillisEstimate" : 1,
        "works" : 5715,
        "advanced" : 711,
        "needTime" : 5003,
        "needYield" : 0,
        "saveState" : 44,
        "restoreState" : 44,
        "isEOF" : 1,
        "transformBy" : {
            "_id" : 0,
            "customer.email" : 1,
            "customer.age" : 1
        },
        "inputStage" : {
            "stage" : "SORT",
            "nReturned" : 711,
            "executionTimeMillisEstimate" : 1,
            "works" : 5715,
            "advanced" : 711,
            "needTime" : 5003,
            "needYield" : 0,
            "saveState" : 44,
            "restoreState" : 44,
            "isEOF" : 1,
            "sortPattern" : {
                "customer.age" : -1
            },
            "memUsage" : 745392,
            "memLimit" : 33554432,
            "inputStage" : {
                "stage" : "SORT_KEY_GENERATOR",
                "nReturned" : 711,
                    "executionTimeMillisEstimate" : 1,
                    "works" : 5003,
                    "advanced" : 711,
                    "needTime" : 4291,
                    "needYield" : 0,
                    "saveState" : 44,
                    "restoreState" : 44,
                    "isEOF" : 1,
                    "inputStage" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                            "$and" : [
                                {
                                    "items.name" : {
                                            "$eq" : "backpack"
                                        }
                                },
                                {
                                        "storeLocation" : {
                                            "$eq" : "Denver"
                                        }
                                }
                            ]
                        },
                        "nReturned" : 711,
                        "executionTimeMillisEstimate" : 1,
                        "works" : 5002,
                        "advanced" : 711,
                        "needTime" : 4290,
                        "needYield" : 0,
                        "saveState" : 44,
                        "restoreState" : 44,
                        "isEOF" : 1,
                        "direction" : "forward",
                        "docsExamined" : 5000
                         }
            }
        }
    }
},
```

输出表明，为了返回`711`条记录，扫描了所有`5000`条记录。它还表明执行从`COLLSCAN`阶段开始，这意味着最初没有索引支持查询中的字段。

为了提高查询性能，可以在集合上创建一个索引。由于查询在筛选条件中使用了两个字段，因此在索引中使用这两个字段。然而，查询还有一个排序规范，根据执行统计，排序是在内存中执行的。为了避免在内存中扫描，将排序字段包含在索引中。

1.  在集合上创建一个复合索引，并包括`items.name`、`storeLocation`和`customer.age`字段。以下查询在`sales`集合上创建了一个复合索引：

```js
db.sales.createIndex(
    {
        "items.name" : 1,
        "storeLocation" : 1,
        "customer.age" : -1
    }
)
```

输出表明索引已正确创建，如下所示：

```js
{
    "createdCollectionAutomatically" : false,
         "numIndexesBefore" : 1,
         "numIndexesAfter" : 2,
         "ok" : 1,
         "$clusterTime" : {
             "clusterTime" : Timestamp(1603246555, 1),
        "signature" : {
            "hash" : BinData(0,"yLQFK4QAJ0ci0M0PzZTex+K73LU="),
            "keyId" : NumberLong("6827475821280624642")
        }
    },
         "operationTime" : Timestamp(1603246555, 1)
}
```

再次执行*步骤 2*中执行的`explain()`查询。以下片段显示了输出的`executionStats`部分：

```js
     "executionStats" : {
          "executionSuccess" : true,
          "nReturned" : 711,
          "executionTimeMillis" : 2,
          "totalKeysExamined" : 711,
          "totalDocsExamined" : 711,
          "executionStages" : {
               "stage" : "PROJECTION_DEFAULT",
               "nReturned" : 711,
               "executionTimeMillisEstimate" : 0,
               "works" : 712,
               "advanced" : 711,
               "needTime" : 0,
               "needYield" : 0,
               "saveState" : 5,
               "restoreState" : 5,
               "isEOF" : 1,
               "transformBy" : {
                    "_id" : 0,
                    "customer.email" : 1,
                    "customer.age" : 1
               },
               "inputStage" : {
                    "stage" : "FETCH",
                    "nReturned" : 711,
                    "executionTimeMillisEstimate" : 0,
                    "works" : 712,
                    "advanced" : 711,
                    "needTime" : 0,
                    "needYield" : 0,
                    "saveState" : 5,
                    "restoreState" : 5,
                    "isEOF" : 1,
                    "docsExamined" : 711,
                    "alreadyHasObj" : 0,
                    "inputStage" : {
                         "stage" : "IXSCAN",
                         "nReturned" : 711,
                         "executionTimeMillisEstimate" : 0,
                         "works" : 712,
                         "advanced" : 711,
                         "needTime" : 0,
                         "needYield" : 0,
                         "saveState" : 5,
                         "restoreState" : 5,
                         "isEOF" : 1,
                         "keyPattern" : {
                              "items.name" : 1,
                              "storeLocation" : 1,
                              "customer.age" : -1
                         },
                         "indexName" : "items.name_1_storeLocation_1_customer.age_-1",
                         "isMultiKey" : true,
                         "multiKeyPaths" : {
                              "items.name" : [
                                   "items"
                              ],
                              "storeLocation" : [ ],
                              "customer.age" : [ ]
                         },
                         "isUnique" : false,
                         "isSparse" : false,
                         "isPartial" : false,
                         "indexVersion" : 2,
                         "direction" : "forward",
                         "indexBounds" : {
                              "items.name" : [
                                   "[\"backpack\", \"backpack\"]"
                              ],
                              "storeLocation" : [
                                   "[\"Denver\", \"Denver\"]"
                              ],
                              "customer.age" : [
                                   "[MaxKey, MinKey]"
                              ]
                         },
                         "keysExamined" : 711,
                         "seeks" : 1,
                         "dupsTested" : 711,
                         "dupsDropped" : 0
                    }
               }
          }
     }
```

从输出结果可以明显看出，执行的第一个阶段是`IXSCAN`，这意味着使用了正确的索引。还要注意到没有排序阶段。这意味着由于`customer.age`字段上的正确索引，不需要进一步排序。顶层执行统计数据显示只扫描了`711`条记录，并且返回了相同数量的记录。这证明了查询被正确优化了。

在这个活动中，你分析了查询的性能统计数据，识别了问题，并创建了正确的索引来解决性能问题。

# 10\. 复制

## 活动 10.01：测试 MongoDB 数据库的灾难恢复过程

**解决方案：**

执行以下步骤完成活动：

1.  创建以下目录：`C:\sale\sale-prod`，`C:\sale\sale-dr`，`C:\sale\sale-ab`和`C:\sale\log`。

注意

对于 Linux 和 macOS，目录名称将类似于`/data/sales/sale-prod`，`/data/sales/sale-dr…`

1.  启动集群节点如下：

```js
start mongod --port 27001 --bind_ip_all --replSet sale-cluster --dbpath C:\sale\sale-prod --logpath C:\sale\log\sale-prod.log --logappend --oplogSize 50
start mongod --port 27002 --bind_ip_all --replSet sale-cluster --dbpath C:\sale\sale-dr --logpath C:\sale\log\sale-dr.log --logappend --oplogSize 50
start mongod --port 27003 --bind_ip_all --replSet sale-cluster --dbpath C:\sale\sale-ab --logpath C:\sale\log\sale-ab.log --logappend --oplogSize 50
```

1.  连接 mongo shell：

```js
 mongo mongodb://localhost:27001/?replicaSet=sale-cluster
```

1.  创建并激活集群配置：

```js
var cfg = { 
    _id : "sale-cluster",
    members : [
        { _id : 0, host : "localhost:27001"},
        { _id : 1, host : "localhost:27002"},
        { _id : 2, host : "localhost:27003", arbiterOnly:true},
        ] 
}
    rs.initiate(cfg)
```

注意

在成功的集群选举后，你应该能够在 shell 提示符上看到`PRIMARY`。

1.  在`sample_mflix`数据库中插入`100`个文档。在主服务器上使用以下脚本创建`sales_data`集合并插入`100`个文档：

```js
use sample_mflix
db.createCollection("sales_data")
for (i=0; i<=100; i++) {
    db.new_sales_data.insert({_id:i, "value":Math.random()})
}
```

1.  通过添加以下命令关闭主服务器：

```js
use admin
db.shutdownServer()
```

1.  通过添加以下命令检查主服务器是否为 DR 实例（首先断开连接，然后重新连接）

```js
rs.isMaster().primary
```

结果应显示`sales_dr`。

1.  使用以下脚本在新的主实例（`sales_dr`）上插入额外的 10 个文档：

```js
use sample_mflix
for (i=101; i<=110; i++) {
    db.new_sales_data.insert({_id:i, "value":Math.random()})
}
```

1.  使用以下命令关闭 DR 数据库和仲裁者：

```js
use admin
db.shutdownServer()
```

1.  确保两者都关闭后，重新启动以前的主服务器：

```js
start mongod --port 27001 --bind_ip_all --replSet sale-cluster --dbpath C:\sale\sale-prod --logpath C:\sale\log\sale-prod.log --logappend --oplogSize 50
```

1.  按照以下步骤重新启动仲裁者：

```js
start mongod --port 27003 --bind_ip_all --replSet sale-cluster --dbpath C:\sale\sale-ab --logpath C:\sale\log\sale-ab.log --logappend --oplogSize 50
```

连接到集群。您应该能够看到在`sales_dr`上插入的 10 个文档，并且`db.new_sales_data.count()`应该只重新运行 100 次。

1.  5 分钟后，按以下方式重新启动 DR 数据库：

```js
start mongod --port 27002 --bind_ip_all --replSet sale-cluster --dbpath C:\sale\sale-dr --logpath C:\sale\log\sale-dr.log --logappend --oplogSize 50
```

1.  在重新启动后，验证`sales_dr 日志`文件中的步骤。在 DR 日志中，您应该能够看到如下消息：

```js
ROLLBACK [rsBackgroundSync] transition to SECONDARY
2019-11-26T15:48:29.538+1000 I  REPL     [rsBackgroundSync] transition to SECONDARY from ROLLBACK
2019-11-26T15:48:29.538+1000 I  REPL     [rsBackgroundSync] Rollback successful.
```

# 11. MongoDB 中的备份和还原

## 活动 11.01：MongoDB 中的备份和还原

**解决方案：**

执行以下步骤以完成活动：

1.  从`mongoexport`开始。删除`--db`选项，因为您在 URI 中提供了它。

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --collection=theaters --out="theaters.csv" --type=csv --sort='{theaterId: 1}'
```

1.  将字段选项添加到`mongoexport`命令

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --fields=theaterId,location --collection=theaters --out="theaters.csv" --type=csv --sort='{theaterId: 1}'
```

1.  将必要的 CSV 选项添加到导入命令，即`type`，`ignoreBlanks`和`headerline`。

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports --type=CSV --headerline --ignoreBlanks --collection=theaters_import --file=theaters.csv 
```

1.  修复`dump`命令的`gzip`选项。

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=./backups –gzip --nsExclude=theaters
```

1.  将`nsExclude`更改为`excludeCollection`：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=./backups –gzip --excludeCollection=theaters
```

1.  在`mongorestore`命令中，修复选项的名称：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --nsFrom="sample_mflix" --nsTo="backup_mflix_backup" --drop ./backups
```

1.  同样在`mongorestore`中，添加`gzip`选项，因为您的转储文件是`gzip`格式：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --nsFrom="sample_mflix" --nsTo="backup_mflix_backup" --gzip --drop ./backups
```

1.  最后，请确保您的命名空间使用通配符进行正确的名称迁移：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --nsFrom="sample_mflix.*" --nsTo="backup_mflix_backup.*" --gzip --drop ./backups
```

1.  最终的`mongoexport`命令应如下所示：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --fields=theaterId,location --collection=theaters --out="theaters.csv" --type=csv --sort='{theaterId: 1}'
```

1.  最终的`mongoimport`命令应如下所示：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports --type=CSV –headerline –ignoreBlanks --collection=theaters_import --file=theaters.csv 
```

1.  最终的`mongodump`命令应如下所示：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=./backups –gzip --excludeCollection=theaters
```

1.  最终的`mongorestore`命令应如下所示：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --nsFrom="sample_mflix.*" --nsTo="backup_mflix_backup.*" --gzip --drop ./backups
```

注意

重要的是要注意，因为`mongoimport`和`mongorestore`都将在数据库中创建新文档，所以您必须使用具有写访问权限的凭据执行这些命令。

# 12.数据可视化

## 活动 12.01：创建销售演示仪表板

**解决方案：**

执行以下步骤以完成活动：

1.  在您可以开始为这个新演示构建图表之前，您必须在应用程序中定义适当的数据来源。按照*练习 12.01*，*使用数据来源*中的步骤，在`sample_supplies`数据库的销售集合上创建一个新的销售数据来源，如下图所示：![图 12.52：创建新的销售数据来源](img/B15507_12_52.jpg)

图 12.52：创建新的销售数据来源

1.  单击“完成”以保存。新的数据来源将显示在列表中，如下图所示：![图 12.53：销售数据来源](img/B15507_12_53.jpg)

图 12.53：销售数据来源

1.  从仪表板中，单击“添加图表”按钮，如下截图所示：![图 12.54：在用户仪表板上单击“添加图表”](img/B15507_12_54.jpg)

图 12.54：在用户仪表板上单击“添加图表”

在“图表生成器”中，选择在*步骤 2*中创建的销售数据来源（即`sample_supplies.sales`），然后选择`Circular`图表类型和`Donut`图表子类型，如下截图所示：

![图 12.55：选择圆形图表类型和甜甜圈图表子类型](img/B15507_12_55.jpg)

图 12.55：选择圆形图表类型和甜甜圈图表子类型

1.  展开`items`数组。这一步很重要，因为销售数据以数组格式存在于 JSON 数据库中。因此，`unwind`函数将为数组中的每个项目创建一个虚拟文档。为此，请将以下 JSON 代码添加到“查询”栏：

```js
[{$unwind:"$items"}]
```

然后单击“应用”按钮，如下截图所示：

![图 12.56：在查询栏中编写展开函数](img/B15507_12_56.jpg)

图 12.56：在查询栏中编写展开函数

1.  下一步是添加一个新的计算字段，即`items.value`。要做到这一点，点击“+添加字段”按钮，并将新字段添加为`items.value = items.price * items.quantity`，如下所示：![图 12.57：添加 items.value 字段](img/B15507_12_57.jpg)

图 12.57：添加 items.value 字段

1.  添加一个过滤器，以便仅考虑来自“丹佛”的商店的商品。从“过滤器”选项卡中，通过仅勾选“丹佛”位置复选框来定义商店位置的新过滤器：![图 12.58：仅从位置列表中选择丹佛](img/B15507_12_58.jpg)

图 12.58：仅从位置列表中选择丹佛

1.  在“编码”选项卡中添加通道。如下图所示，将字段`items.name`拖入“标签”通道。从“排序方式”下拉菜单中选择`VALUE`，并将结果限制为`10`个。这将把我们的圆环图分成 10 个部分。类似地，将`items.value`（新计算字段）拖入“弧”通道，并从“聚合”下拉菜单中选择`SUM`函数：![图 12.59：将 items.value 拖入弧道并选择 SUM 函数](img/B15507_12_59.jpg)

图 12.59：将 items.value 拖入弧道并选择 SUM 函数

1.  图表应该出现在屏幕的右侧，如下所示：![图 12.60：最终图表](img/B15507_12_60.jpg)

图 12.60：最终图表

1.  将图表名称编辑为“丹佛销售（百万美元）”，如下所示：![图 12.61：编辑图表标题](img/B15507_12_61.jpg)

图 12.61：编辑图表标题

1.  编辑图表标签。从“自定义”选项卡，点击启用“数据值标签”，如下所示：![图 12.62：自定义数据标签](img/B15507_12_62.jpg)

图 12.62：自定义数据标签

1.  接下来，从“数字格式”下拉菜单中选择“自定义”，最多保留`2`位小数，如下所示：![图 12.63：自定义图表格式](img/B15507_12_63.jpg)

图 12.63：自定义图表格式

1.  图表将显示正确的标题和标签格式，如下图所示：![图 12.64：最终丹佛销售图表](img/B15507_12_64.jpg)

图 12.64：最终丹佛销售图表

结果相当不言自明。如预期的那样，笔记本电脑的销售价值接近 200 万美元，居销售榜首，并且是销售报告中价值最高的商品。销售额排名第二的商品是背包，价值仅 25 万美元。

活动现在已经完成。在仅 10 个简单的步骤中，您已经能够为科罗拉多州丹佛市商店的商品创建一份销售报告。您的图表构建现在已经完成，可以保存在您的仪表板上。在这里学到的经验可以被学生和专业人士应用，以使用真实数据进行演示。
