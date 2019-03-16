### Maven源码分析

###### Maven启动入口方法：MavenCli  plexus读取m2.conf，反射invoke 运行MavenCli的main方法
```html
int result = cli.doMain( new CliRequest( args, classWorld ) );

public int doMain( CliRequest cliRequest )
{
    PlexusContainer localContainer = null;
    try
    {
        initialize( cliRequest ); //初始化，工作目录变量，maven.home设置
        cli( cliRequest );        //解析commandLine参数
        properties( cliRequest ); //装载systemProperties、userProperties,buildProperties
        logging( cliRequest );    //日志，命令行传参数，控制日志
        version( cliRequest );    //maven版本信息输出 
        localContainer = container( cliRequest );  //创建Ioc容器，并获取maven等必要实例
        commands( cliRequest );  //打印部分状态信息
        configure( cliRequest ); //
        toolchains( cliRequest );
        populateRequest( cliRequest ); //装配请求对象
        encryption( cliRequest );      //deploy setting.xml 密码
        repository( cliRequest );      //是否使用LegacyLocalRepository仓库
        return execute( cliRequest );
    }
}
```


---
###### org.apache.maven.DefaultMaven#doExecute(MavenExecutionRequest)  
```html
1. 初始化属性

2. 验证本地仓库目录是否存在或是否可以访问
validateLocalRepository(request );

3. 创建RepositorySystemSession，带有authentication,mirror, proxy和其它系统环境信息
newRepositorySession(request ) ;

4.创建MavenSession，带有容器，RepositorySystemSession，请求对象，返回结果对象
newMavenSession( container, repoSession, request, result );

5.执行AbstractLifecycleParticipant.afterSessionStart(session)

6. 获取依赖的项目，检查是否有pom错误
projects= getProjectsForMavenReactor( session );
validateProjects(projects );

7.创建ProjectDependencyGraph，带有剪裁反应堆，如：-pl，-rf


8.创建ReactorReader，用于查找maven反应堆模块
reactorWorkspace= container.lookup( WorkspaceReader.class, ReactorReader.HINT );

9.执行AbstractLifecycleParticipant.afterProjectsRead(session)

10.创建ProjectDependencyGraph，不带剪裁

11.执行LifecycleStarter.start()，开始执行生命周期任务
lifecycleStarter.execute(session );
```
