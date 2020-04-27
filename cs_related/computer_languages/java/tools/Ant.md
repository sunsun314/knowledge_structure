# 工具Ant相关重要内容摘录

## 一个简单的例子

```
<?xml version="1.0"?>
<project name="fax" basedir="." default="build">
   <property name="src.dir" value="src"/>
   <property name="web.dir" value="war"/>
   <property name="build.dir" value="${web.dir}/WEB-INF/classes"/>
   <property name="name" value="fax"/>

   <path id="master-classpath">
      <fileset dir="${web.dir}/WEB-INF/lib">
         <include name="*.jar"/>
      </fileset>
      <pathelement path="${build.dir}"/>
   </path>

   <target name="build" description="Compile source tree java files">
      <mkdir dir="${build.dir}"/>
      <javac destdir="${build.dir}" source="1.5" target="1.5">
         <src path="${src.dir}"/>
         <classpath refid="master-classpath"/>
      </javac>
   </target>

   <target name="clean" description="Clean output directories">
      <delete>
         <fileset dir="${build.dir}">
            <include name="**/*.class"/>
         </fileset>
      </delete>
   </target>
   
   <target name="build-jar">
   <jar destfile="${web.dir}/lib/util.jar" basedir="${build.dir}/classes"
      includes="faxapp/util/**" excludes="**/Test.class">
      <manifest>
         <attribute name="Main-Class" value="com.tutorialspoint.util.FaxUtil"/>
      </manifest>
   </jar>
</target>
</project>
```
**介绍:**
+ property如其名参数类型，设定了一些后续使用的参数名称和参数的实际值
+ path 其实是指定了一组文件集合，可以在后续中被用来进行任何类型的操作
+ target 这里的target和maven生成的target包有关键性的区别，这里的意思其实是command，然后其中的内容定义了这个command处理的真正内容，上面例子中其实是定义了build和clean两个命令

## 总结整理

Ant这个工具的使用比较奇怪，如果发生执行的过程中报错的情况，其实是应该从target也就是入口命令逐步去找具体的那个target命令，然后根据命令执行的内容在倒回去找具体什么步骤出了问题  

相比maven的自动化来说，Ant的操作比较本质，但也比较原始，有点明白为啥会逐渐淘汰了。