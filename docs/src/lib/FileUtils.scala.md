## Generic file utilities

```scala
package laughedelic.literator.lib

import java.io._
import java.nio.file.Path
```

This type represents a node of file hierarchy tree

```scala
case class FileNode(f: File, t: List[FileNode]) {
  import FileUtils._

  def link(base: File): String =
    if (f.isSource) s"[${f.name}][${f.relativePath(base)}]"
    else f.name

  def listTree(base: File): List[String] = {
    s"+ ${link(base)}" ::
    t.flatMap { i: FileNode =>
      i.listTree(base).map { str => s"  ${str}" }
    }
  }

  override def toString: String = listTree(f).mkString("\n")
}
```

Let's extend `File` type with some useful functions

```scala
object FileUtils {

  // just an alias:
  def file(path: String): File = new File(path)

  implicit class FileOps(file: File) {
```

Returning path of `file` relatively to `base`

```scala
    def relativePath(base: File): Path = {
      val b = if (base.isDirectory) base.getCanonicalFile
              else base.getCanonicalFile.getParentFile
      b.toPath.relativize(file.getCanonicalFile.toPath)
    }
```

Name (last part after `/`) and extension

```scala
    def name: String = file.getCanonicalFile.getName
    def ext: String = if (file.isDirectory) "" else file.name.split("\\.").last
```

Traversing recursively and listing all _files_ passing the filter

```scala
    def getFileList(filter: (File => Boolean) = (_ => true)): List[File] =
      if (file.isDirectory) file.listFiles.toList.flatMap(_.getFileList(filter))
      else if (filter(file)) List(file) else List()
```

Traversing recursively and building file hierarchy tree

```scala
    def getFileTree(filter: (File => Boolean) = (_ => true)): Option[FileNode] =
      if (!filter(file)) None
      else Some(FileNode(file,
            if (file.isDirectory) file.listFiles.toList.map(_.getFileTree(filter)).flatten
            else List()
           ))
```

Read the contents of the file

```scala
    def read: String = scala.io.Source.fromFile(file, "UTF-8").mkString
```

Just writing to the file

```scala
    def write(text: String) = {
      Some(new PrintWriter(file)).foreach{p => p.write(text); p.close}
    }

  }
}

```


------

### Index

+ scala
  + lib
    + [FileUtils.scala][lib/FileUtils.scala]
    + [LanguageMap.scala][lib/LanguageMap.scala]
    + [LiteratorParsers.scala][lib/LiteratorParsers.scala]
    + [package.scala][lib/package.scala]
  + plugin
    + [LiteratorPlugin.scala][plugin/LiteratorPlugin.scala]

[lib/FileUtils.scala]: FileUtils.scala.md
[lib/LanguageMap.scala]: LanguageMap.scala.md
[lib/LiteratorParsers.scala]: LiteratorParsers.scala.md
[lib/package.scala]: package.scala.md
[plugin/LiteratorPlugin.scala]: ../plugin/LiteratorPlugin.scala.md
[Readme.md]: ../Readme.md.md