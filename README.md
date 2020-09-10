# MongoDB中文手册\|官方文档中文版

## 项目介绍

MongoDB是专为可扩展性，高性能和高可用性而设计的数据库。它可以从单服务器部署扩展到大型、复杂的多数据中心架构。利用内存计算的优势，MongoDB能够提供高性能的数据读写操作。 MongoDB的本地复制和自动故障转移功能使您的应用程序具有企业级的可靠性和操作灵活性。

本项目为MongoDB官方文档的中文版，与[官方文档](https://docs.mongodb.com/manual/)保持同步。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x5185;&#x5BB9;</th>
      <th style="text-align:left">&#x8BF4;&#x660E;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x672C;&#x624B;&#x518C;&#x6587;&#x6863;&#x7248;&#x672C;</td>
      <td style="text-align:left">&#x57FA;&#x4E8E;4.2&#x7248;&#x672C;&#xFF0C;&#x4E0D;&#x65AD;&#x4E0E;&#x6700;&#x65B0;&#x7248;&#x4FDD;&#x6301;&#x540C;&#x6B65;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7EF4;&#x62A4;&#x5730;&#x5740;</td>
      <td style="text-align:left"><a href="https://github.com/mongodb-china/MongoDB-CN-Manual">mongodb-china Github</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x4E2D;&#x6587;&#x6587;&#x6863;&#x5728;&#x7EBF;&#x9605;&#x8BFB;</td>
      <td
      style="text-align:left"><a href="https://docs.mongoing.com/">MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;&#x6587;&#x6863;&#x5728;&#x7EBF;&#x9605;&#x8BFB;</a> ;
        <a
        href="https://docs.jinmu.info/MongoDB-Manual-zh/">&#x4E0A;&#x6D77;&#x9526;&#x6728;&#x6587;&#x6863;&#x5728;&#x7EBF;&#x9605;&#x8BFB;</a>
          </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5728;&#x7EBF;&#x9605;&#x8BFB;&#x95EE;&#x9898;&#x62A5;&#x544A;</td>
      <td
      style="text-align:left">&#x5982;&#x679C;&#x60A8;&#x53D1;&#x73B0;&#x95EE;&#x9898;&#xFF0C;&#x8BF7;&#x5728;
        <a
        href="https://github.com/mongodb-china/MongoDB-CN-Manual/issues">MongoDB-Manual-zh/issues</a>&#x4E0A;&#x63D0; issue</td>
    </tr>
    <tr>
      <td style="text-align:left"><a href="https://github.com/mongodb-china/MongoDB-CN-Manual/blob/master/List-of-contributors.md">&#x6587;&#x6863;&#x7FFB;&#x8BD1;&#x8D21;&#x732E;&#x8005;&#x540D;&#x5355;</a>
      </td>
      <td style="text-align:left"><a href="https://mongoing.com/">MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;</a>&amp;
        <a
        href="http://www.jinmuinfo.com/">&#x4E0A;&#x6D77;&#x9526;&#x6728;</a>&#x5DF2;&#x5B8C;&#x6210;&#x5927;&#x90E8;&#x5206;&#x7FFB;&#x8BD1;&#xFF0C;&#x6B22;&#x8FCE;&#x66F4;&#x591A;mongoers&#x52A0;&#x5165;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5982;&#x4F55;&#x52A0;&#x5165;&#x6587;&#x6863;&#x7FFB;&#x8BD1;</td>
      <td
      style="text-align:left">&#x70B9;&#x51FB;<a href="https://github.com/mongodb-china/MongoDB-CN-Manual/blob/master/Document-translation-claim-list.md">&#x6587;&#x6863;&#x7FFB;&#x8BD1;&#x8BA4;&#x9886;&#x5217;&#x8868;</a>&#x52A0;&#x5165;&#x6587;&#x6863;&#x7FFB;&#x8BD1;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x6587;&#x6863;&#x7FFB;&#x8BD1;&#x89C4;&#x8303;</td>
      <td style="text-align:left"><a href="https://github.com/mongodb-china/MongoDB-CN-Manual/blob/master/CONTRIBUTING.md">&#x8D21;&#x732E;&#x6307;&#x5357;</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x52A0;&#x5165;&#x7FFB;&#x8BD1;&#x6743;&#x76CA;</td>
      <td style="text-align:left">
        <p>&#x7FFB;&#x8BD1;&#x5185;&#x5BB9;&#x5C06;&#x7531;MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;&#x4E13;&#x5BB6;&#x8FDB;&#x884C;&#x5BA1;&#x6838;&#xFF0C;</p>
        <p>&#x5BA1;&#x6838;&#x901A;&#x8FC7;&#x540E;&#x5C06;&#x4FDD;&#x7559;&#x7F72;&#x540D;&#x6743;&#x53D1;&#x5E03;&#x5230;&#x672C;&#x624B;&#x518C;&#x53CA;MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;&#x5FAE;&#x4FE1;&#x5185;&#x5BB9;&#x5E73;&#x53F0;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>

## LICENSE

  本项目为署名-非商业性使用-相同方式共享 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)

## [MongoDB中文社区](https://mongoing.com/)

![MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;&#x2014;MongoDB&#x7231;&#x597D;&#x8005;&#x6280;&#x672F;&#x4EA4;&#x6D41;&#x5E73;&#x53F0;](https://mongoing.com/wp-content/uploads/2020/09/6de8a4680ef684d-2.png)



