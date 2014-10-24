---
layout: post
title: JavaEE7 批处理和魔兽世界
category : JavaEE
tags : [Java, WOW, JavaEE]
---
{% include JB/setup %}

本文的目的主要是结合 MMORPG 游戏 [魔兽世界](http://eu.battle.net/wow/en/ "魔兽世界") 展开说明一下  [JSR-352](https://jcp.org/en/jsr/detail?id=352 "JSR-352") 在实际开发中的作用。

因为 [JSR-352](https://jcp.org/en/jsr/detail?id=352 "JSR-352") 是 Java 的一个新特性。所以很多人对如何使用它还不是很了解，不明白该特性用在哪。幸好本文中的例子会让你对它的应用场景有一个更深入的了解。

## 简介 ##

[魔兽世界](http://eu.battle.net/wow/en/ "魔兽世界") 在全世界有八百万玩家。这些玩家主要集中在四个区域：美国（**US**）、欧洲（**EU**）、中国和韩国。每个服务区域有很多叫做 ***Realm*** 的服务群，连接到这些服务器你就可以开始于游戏了。本文中，我们主要看下美国和欧洲区。

![魔兽世界拍卖行]({{ site.JB.POST_IMG_PATH }}/20141023143536.jpg)

在这个游戏中允许你通过 ***Auction House*** 买卖游戏中的叫做 ***Item*** 的东西。每一个 ***Realm*** 中有两个 ***Auction House***。平均每一个 ***Realm*** 大约有 70000 件 ***Item*** 正在被交易。我们再看看这些数字：

* 512 个 ***Realm *** (美国和欧洲)
* 每个 ***Realm *** 约 70K 件 ***Item***
* 总量超过 35M 件 ***Item***

### 数据 ###

[魔兽世界](http://eu.battle.net/wow/en/ "魔兽世界") 中另一件很酷的事情是它的开发者们为获取这些游戏中的信息提供了一组 REST 接口，包括 ***Auction House***。[点击](http://blizzard.github.io/api-wow-docs/) 查看全部 API。

获取 ***Auction House*** 的数据主要分两步。首先我们要查询 ***Auction House Realm*** 接口获取一个 JSON 文件的地址列表。然后访问这个地址，这个地址文件包含着全部的商品信息。例如：

[http://eu.battle.net/api/wow/auction/data/aggra-portugues](http://eu.battle.net/api/wow/auction/data/aggra-portugues)

## 开始动手 ##

我们的目标就是要开发一个应用来下载这些 ***Auction House*** 的数据并进行处理分析。最后建立一个 ***Item*** 价格历史走向图。或许通过这个图预测未来的价格走向并在合适的时间买入或者卖出。

### 准备 ###

准备阶段我们需要了解下以下几项东西：

* [JavaEE 7](http://docs.oracle.com/javaee/7/tutorial/doc/home.htm)
* [Angular JS](http://angularjs.org/)
* [Angular ng-grid](http://angular-ui.github.io/ng-grid/)
* [UI BootStrap](http://angular-ui.github.io/bootstrap/)
* [Google Chart](https://developers.google.com/chart/)
* [Wildfly](http://www.wildfly.org/)

### 任务 ###

主要任务会被通过 [JSR-352](https://jcp.org/en/jsr/detail?id=352 "JSR-352") 的批处理执行。一个任务是一批处理的集合。它们通过特定的任务描述语言集成到一起。在 [JSR-352](https://jcp.org/en/jsr/detail?id=352 "JSR-352") 中，任务是操作的容器。

现在我们主要包括以下几个任务：

* **准备** - 创建所有需要的数据。罗列 ***Realm***，创建必须的文件夹
* **文件** - 查询 ***Realm*** 检查需要处理的新文件
* **处理** - 下载文件，处理分析数据

## 关于代码 ##

### 后台 - JavaEE 7 - Java 8 ###

大部分代码都是后台代码。我们需要 [JSR-352](https://jcp.org/en/jsr/detail?id=352 "JSR-352") 的批处理，同时也需要一些 Java EE 提供的其它技术： [JPA](https://jcp.org/en/jsr/detail?id=338)、[JAX-RS](https://jax-rs-spec.java.net/)、[CDI](http://cdi-spec.org/)、[JSON-P](https://jsonp.java.net/) 等。

因为准备任务只是初始化应用的资源，此处直接跳过开始最有趣的部分了。

### 文件任务 ###

文件任务就是 [`AbstractBatchlet`](http://docs.oracle.com/javaee/7/api/javax/batch/api/AbstractBatchlet.html) 的一个实现。`Batchlet` 是 `Batch` 规范中最简单的实现方式。它面向任务，只被调用一次，直接执行并返回状态码。这种类型在处理一批非面向对象的任务时非常有用，例如执行命令或者文件传输。在我们这，我们用它遍历 ***Realm***，发送 REST 请求并取得包含需要处理数据的文件地址。代码如下：

{% highlight java linenos %}
@Named
public class LoadAuctionFilesBatchlet extends AbstractBatchlet {
    @Inject
    private WoWBusiness woWBusiness;
 
    @Inject
    @BatchProperty(name = "region")
    private String region;
    @Inject
    @BatchProperty(name = "target")
    private String target;
 
    @Override
    public String process() throws Exception {
        List<Realm> realmsByRegion = woWBusiness.findRealmsByRegion(Realm.Region.valueOf(region));
        realmsByRegion.parallelStream().forEach(this::getRealmAuctionFileInformation);
 
        return "COMPLETED";
    }
 
    void getRealmAuctionFileInformation(Realm realm) {
        try {
            Client client = ClientBuilder.newClient();
            Files files = client.target(target + realm.getSlug())
                                .request(MediaType.TEXT_PLAIN).async()
                                .get(Files.class)
                                .get(2, TimeUnit.SECONDS);
 
            files.getFiles().forEach(auctionFile -> createAuctionFile(realm, auctionFile));
        } catch (Exception e) {
            getLogger(this.getClass().getName()).log(Level.INFO, "Could not get files for " + realm.getRealmDetail());
        }
    }
 
    void createAuctionFile(Realm realm, AuctionFile auctionFile) {
        auctionFile.setRealm(realm);
        auctionFile.setFileName("auctions." + auctionFile.getLastModified() + ".json");
        auctionFile.setFileStatus(FileStatus.LOADED);
 
        if (!woWBusiness.checkIfAuctionFileExists(auctionFile)) {
            woWBusiness.createAuctionFile(auctionFile);
        }
    }
}
{% endhighlight %}

这段代码非常酷的用了 Java 8。通过 `parallelStream()` 非常容易的调用了 REST 请求。你可以尝试下载代码并把 `parallelStream()` 替换成 `stream()` 再运行一遍。在我的机器上通过 `parallelStream()` 能获得获得 5 - 6 倍的效率提升。

因为美国和欧洲的 ***Realm*** 需要调用不同的 REST 结点，所以此处最好把它们做一下分区。分区后就可以在多线程环境下执行它们。可以每个分区一个线程。现在，我们把它们分成两个。

为了完成任务定义我们还需要一个 XML 文件。它需要被放在 `META-INF/batch-jobs` 目录下。`files-job.xml` 文件如下：

{% highlight xml linenos %}
<job id="loadRealmAuctionFileJob" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="loadRealmAuctionFileStep">
        <batchlet ref="loadAuctionFilesBatchlet">
            <properties>
                <property name="region" value="#{partitionPlan['region']}"/>
                <property name="target" value="#{partitionPlan['target']}"/>
            </properties>
        </batchlet>
        <partition>
            <plan partitions="2">
                <properties partition="0">
                    <property name="region" value="US"/>
                    <property name="target" value="http://us.battle.net/api/wow/auction/data/"/>
                </properties>
                <properties partition="1">
                    <property name="region" value="EU"/>
                    <property name="target" value="http://eu.battle.net/api/wow/auction/data/"/>
                </properties>
            </plan>
        </partition>
    </step>
</job>
{% endhighlight %}

在 `files-job.xml` 文件中我们在 `batchlet` 结点中定义了我们自己的 [`Batchlet`](http://docs.oracle.com/javaee/7/api/javax/batch/api/Batchlet.html) 。为了分区我们还需要定义 `partition` 结点，为每个 `plan` 定义不同的 `properties`。`properties` 中的值稍后会被通过 `#{partitionPlan['region']}` 和 `#{partitionPlan['target']}` 绑定到 `LoadAuctionFilesBatchlet` 类中。这是非常简单的一种绑定方式，也只能适用于非常简单的属性绑定。

### 处理任务 ###

现在我们需要处理 ***Realm Auction Data*** 文件了。从前一个任务获取到的信息中，我们可以下载到该数据文件。它的结构大致是这样的：

{% highlight json linenos %}
{
    "realm": {
        "name": "Grim Batol",
        "slug": "grim-batol"
    },
    "alliance": {
        "auctions": [
            {
                "auc": 279573567,            // Auction Id
                "item": 22792,               // Item for sale Id
                "owner": "Miljanko",         // Seller Name
                "ownerRealm": "GrimBatol",   // Realm
                "bid": 3800000,              // Bid Value
                "buyout": 4000000,           // Buyout Value
                "quantity": 20,              // Numbers of items in the Auction
                "timeLeft": "LONG",          // Time left for the Auction
                "rand": 0,
                "seed": 1069994368
            },
            {
                "auc": 278907544,
                "item": 40195,
                "owner": "Mongobank",
                "ownerRealm": "GrimBatol",
                "bid": 38000,
                "buyout": 40000,
                "quantity": 1,
                "timeLeft": "VERY_LONG",
                "rand": 0,
                "seed": 1978036736
            }
        ]
    },
    "horde": {
        "auctions": [
            {
                "auc": 278268046,
                "item": 4306,
                "owner": "Thuglifer",
                "ownerRealm": "GrimBatol",
                "bid": 570000,
                "buyout": 600000,
                "quantity": 20,
                "timeLeft": "VERY_LONG",
                "rand": 0,
                "seed": 1757531904
            },
            {
                "auc": 278698948,
                "item": 4340,
                "owner": "Celticpala",
                "ownerRealm": "Aggra(Português)",
                "bid": 1000000,
                "buyout": 1000000,
                "quantity": 10,
                "timeLeft": "LONG",
                "rand": 0,
                "seed": 0
            }
        ]
    }
}
{% endhighlight %}

这个文件中有来自它下载的 ***Realm*** 中的 ***Auction House*** 列表。在每一条记录中我们可以得知出售中的物品名、价格、来源和剩余时间等信息。***Auction House***分为两类：***Alliance*** 和 ***Horde***。

对于 `process-job` 而言我们需要读取这个 JSON 文件，转换数据并把它保存到数据库中。这可以通过 `Chunk` 方式处理。`Chunk` 是一种 ETL (Extract – Transform – Load) 方式的处理过程，非常适合处理大量数据。它在一个事务中会不停的读取记录并为之创建任务准备提交。记录通过 [`ItemReader`](http://docs.oracle.com/javaee/7/api/javax/batch/api/chunk/ItemReader.html) 读取并提交给 [`ItemProcessor`](http://docs.oracle.com/javaee/7/api/javax/batch/api/chunk/ItemProcessor.html)，等到数量达到提交阀值时，通过 [`ItemWriter`](http://docs.oracle.com/javaee/7/api/javax/batch/api/chunk/ItemWriter.html) 写出，然后事务结束。

#### ItemReader ####

真实文件过大，我们不可能把它们整个加载到内存中，所以我们通过 JSON-P 按照流式处理这些数据。

{% highlight java linenos %}
@Named
public class AuctionDataItemReader extends AbstractAuctionFileProcess implements ItemReader {
    private JsonParser parser;
    private AuctionHouse auctionHouse;
 
    @Inject
    private JobContext jobContext;
    @Inject
    private WoWBusiness woWBusiness;
 
    @Override
    public void open(Serializable checkpoint) throws Exception {
        setParser(Json.createParser(openInputStream(getContext().getFileToProcess(FolderType.FI_TMP))));
 
        AuctionFile fileToProcess = getContext().getFileToProcess();
        fileToProcess.setFileStatus(FileStatus.PROCESSING);
        woWBusiness.updateAuctionFile(fileToProcess);
    }
 
    @Override
    public void close() throws Exception {
        AuctionFile fileToProcess = getContext().getFileToProcess();
        fileToProcess.setFileStatus(FileStatus.PROCESSED);
        woWBusiness.updateAuctionFile(fileToProcess);
    }
 
    @Override
    public Object readItem() throws Exception {
        while (parser.hasNext()) {
            JsonParser.Event event = parser.next();
            Auction auction = new Auction();
            switch (event) {
                case KEY_NAME:
                    updateAuctionHouseIfNeeded(auction);
 
                    if (readAuctionItem(auction)) {
                        return auction;
                    }
                    break;
            }
        }
        return null;
    }
 
    @Override
    public Serializable checkpointInfo() throws Exception {
        return null;
    }
 
    protected void updateAuctionHouseIfNeeded(Auction auction) {
        if (parser.getString().equalsIgnoreCase(AuctionHouse.ALLIANCE.toString())) {
            auctionHouse = AuctionHouse.ALLIANCE;
        } else if (parser.getString().equalsIgnoreCase(AuctionHouse.HORDE.toString())) {
            auctionHouse = AuctionHouse.HORDE;
        } else if (parser.getString().equalsIgnoreCase(AuctionHouse.NEUTRAL.toString())) {
            auctionHouse = AuctionHouse.NEUTRAL;
        }
 
        auction.setAuctionHouse(auctionHouse);
    }
 
    protected boolean readAuctionItem(Auction auction) {
        if (parser.getString().equalsIgnoreCase("auc")) {
            parser.next();
            auction.setAuctionId(parser.getLong());
            parser.next();
            parser.next();
            auction.setItemId(parser.getInt());
            parser.next();
            parser.next();
            parser.next();
            parser.next();
            auction.setOwnerRealm(parser.getString());
            parser.next();
            parser.next();
            auction.setBid(parser.getInt());
            parser.next();
            parser.next();
            auction.setBuyout(parser.getInt());
            parser.next();
            parser.next();
            auction.setQuantity(parser.getInt());
            return true;
        }
        return false;
    }
 
    public void setParser(JsonParser parser) {
        this.parser = parser;
    }
}
{% endhighlight %}

为了开始 JSON 解析，我们需要调用 `Json.createParser` 并把文件的输入流传给它。为了读取结点我们只需要调用 `hasNext()` 和 `next()` 方法。它返回 `JsonParser.Event` 对象允许我们获取当前解析在流中所处的位置。所有的结点通过 Batch API 中 [`ItemReader`](http://docs.oracle.com/javaee/7/api/javax/batch/api/chunk/ItemReader.html) 的 `readItem()` 读取并返回。当没有结点可以读取时，返回 `null` 来结束本次流程。需要注意的是我们同时实现了来自 [`ItemReader`](http://docs.oracle.com/javaee/7/api/javax/batch/api/chunk/ItemReader.html) 中的 `open` 和 `close` 方法。它们主要用来进行资源初始化和回收。只会执行一次。

#### ItemProcessor ####

`ItemProcessor` 是可选的。它主要用来对我们读取的数据进行下转换。此处我们用它来为 ***Auction*** 添加额外信息。

{% highlight java linenos %}
@Named
public class AuctionDataItemProcessor extends AbstractAuctionFileProcess implements ItemProcessor {
    @Override
    public Object processItem(Object item) throws Exception {
        Auction auction = (Auction) item;
 
        auction.setRealm(getContext().getRealm());
        auction.setAuctionFile(getContext().getFileToProcess());
 
        return auction;
    }
}
{% endhighlight %}

#### ItemWriter ####

最后我们只需要把这些数据写入到数据库中：

{% highlight java linenos %}
@Named
public class AuctionDataItemWriter extends AbstractItemWriter {
    @PersistenceContext
    protected EntityManager em;
 
    @Override
    public void writeItems(List<Object> items) throws Exception {
        items.forEach(em::persist);
    }
}
{% endhighlight %}

在我的机器上处理 70K 的数据大约花费 20 秒的时间。

为了定义这个任务我们还需要在 `process-job.xml` 文件中添加以下：

{% highlight xml linenos %}
<step id="processFile" next="moveFileToProcessed">
    <chunk item-count="100">
        <reader ref="auctionDataItemReader"/>
        <processor ref="auctionDataItemProcessor"/>
        <writer ref="auctionDataItemWriter"/>
    </chunk>
</step>
{% endhighlight %}

在 `item-count` 属性中我们定义了每次处理的数量上限。它的意思是每 100 条数据就提交一次事务。这对维持小事务和建立恢复点是非常有好处的。这在我们需要重启任务时不需要把全部数据再处理一遍。

### 运行 ###

为了运行任务我们还需要一个 [`JobOperator`](http://docs.oracle.com/javaee/7/api/javax/batch/operations/JobOperator.html)。[`JobOperator`](http://docs.oracle.com/javaee/7/api/javax/batch/operations/JobOperator.html) 提供了任务执行过程中的全部对外接口和运行指令。

我们只需要执行：
{% highlight java linenos %}
JobOperator jobOperator = BatchRuntime.getJobOperator();
jobOperator.start("files-job", new Properties());
{% endhighlight %}

需要注意的是我们把定义任务的 XML 文件去除后缀名后传给了 [`JobOperator`](http://docs.oracle.com/javaee/7/api/javax/batch/operations/JobOperator.html)。

## 下一步 ##

目前为止，我们还需要合并分析这些数据并把它们展示在一个 WEB 页面中。敬请期待后续内容。。。

## 资源 ##

你可以通过 GitHub 获取本文代码并在 Wildfly 中部署它：

[World of Warcraft Auctions](https://github.com/radcortez/wow-auctions)

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://www.radcortez.com/java-ee-7-batch-processing-and-world-of-warcraft-part-1/ "JAVA EE 7 BATCH PROCESSING AND WORLD OF WARCRAFT – PART 1")