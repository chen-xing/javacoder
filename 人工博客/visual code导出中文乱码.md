# visual code导出中文乱码

Local Render Settings:

- `plantuml.java`: Java executable location.
- `plantuml.commandArgs`: commandArgs allows you add command arguments to java command, such as `-DPLANTUML_LIMIT_SIZE=8192`.
- `plantuml.jar`: Alternate plantuml.jar location. Leave it blank to use integrated jar.
- `plantuml.jarArgs`: jarArgs allows you add arguments to plantuml.jar, such as `-config plantuml.config`.
- `plantuml.includepaths`: Specifies the include paths besides source folder and the `diagramsRoot`.

Export Settings:

- `plantuml.diagramsRoot`: Specifies where all diagram files located (relative to workspace folder).
- `plantuml.exportOutDir`: Exported workspace diagrams will be organized in this directory (relative path to workspace folder).
- `plantuml.fileExtensions`: File extensions that find to export. Especially in workspace settings, you may add your own extensions so as to export diagrams in source code files, like ".java".
- `plantuml.exportFormat`: format to export. default is not set, user may pick one format everytime exports. You can still set a format for it if you don't want to pick.
- `plantuml.exportSubFolder`: export diagrams to a folder which has same name with host file.
- `plantuml.exportConcurrency`: decides concurrency count when export multiple diagrams.
- `plantuml.exportMapFile`: Determine whether export image map (.cmapx) file when export.





```
{
     “ plantuml.jar ”：“ somepath \\ plantuml.jar ”，
     “ plantuml.commandArgs ”：[
     “- Dfile.encoding=UTF-8 ”
    ]
}
```



```
{
   "plantuml.commandArgs": [
      "-Dfile.encoding=UTF-8"
   ], 
   "plantuml.jarArgs": [
      "-charset UTF-8"
   ]
}
```

