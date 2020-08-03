# Clean
`msbuild path/to/your.sln -target:Clean`

# 编译Release版
`msbuild path/to/your.sln /property:Configuration=Release`

# 编译输出更详细信息
`msbuild path/to/your.sln /property:Configuration=Release -v:d`
`msbuild path/to/your.sln /property:Configuration=Release -v:diag`