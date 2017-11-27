##nodejs __dirname 与 process.cwd(); 的区别
cwd() 是当前执行node命令时候的文件夹地址    
__dirname 是被执行的js 文件的地址   
前者进程发起时的位置，后者是脚本的位置，两者可能是不一致的。比如，node ./code/program.js，对于process.cwd()来说，返回的是当前目录（.）；      
对于__dirname来说，返回是脚本所在目录，即./code/program.js。