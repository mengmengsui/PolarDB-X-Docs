# 编码风格

## 编程风格

贡献者提交的代码，均应遵循相应组件的编程风格，详情请参见各组件仓库中的代码风格指南。

## 代码设计和审查

关于代码审查，阿里云同样遵循[Google开源代码审查](https://github.com/google/eng-practices/blob/master/review/index.md)中的思想和规则。

在提交代码审查之前，请进行单元测试。单元测试应与代码修改一并提交。

除了代码审查之外，本文档还提供了整个高质量开发周期的说明，包括设计、实施、测试、文档开发和准备代码审查。在开发过程中，关键步骤存在很多问题，比如关于设计、功能、复杂性、测试、命名、文档开发以及代码审查的问题。本文档总结了如下代码审查规则。

*进行代码审查时，应该确保：*

* *代码设计完善。*
* *代码的功能性对使用者友好。*
* *任何UI改变都合理美观。*
* *任何并行编程都安全完成。*
* *代码不应“过分”复杂。*
* *开发人员不该实现一个现在不用而未来可能需要的功能。*
* *代码具有适当的单元测试。*
* *测试经过完善的设计。*
* *命名清晰。*
* *代码注释清晰且有用，主要解释为什么而非是什么。*
* *有适当的文档对代码进行说明。*
* *代码符合阿里云代码的风格指南。*