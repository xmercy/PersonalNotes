# VS中配置多个项目之间的依赖编译

## 应用场景

解决方案下有三个项目A，B，C，A项目为公共模块，生成静态库，供B、C两项目使用，即B、C项目都依赖于A项目。一旦改动A，需要先生成A，再生成B、C，才能保证B、C使用A最新的代码逻辑。现在，我们希望在改动了A之后，不需要手动生成A，在生成B或者C的时候，自动的先生成A，在生成B。

## 解决方案

右键B项目选择：“生成依赖项”->“项目依赖项”，在该项目依赖的项目A前面打上勾即可，在生成B的时候会自动生成A，C项目做相同设置即可。



