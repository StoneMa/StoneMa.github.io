---
title: 考试院.net项目技术总结
date: 2018-07-10 09:29:08
tags: [.net技术,web开发]
categories: 开发笔记
---
开发考试院笔迹鉴别系统的过程中遇到了很多的问题，也学到了很多，将其记录下来。以供今后回顾。
**Warning:**项目构建和项目打开的时候一定要使用解决方法，不要使用网站！一定要使用解决方法，不要使用网站！一定要使用解决方法，不要使用网站！重要的事情说三遍！！！网站项目是给静态web项目用的。使用解决方法打开项目，才能每次生成项目的时候把dll更新到最新的状态。

# 数据库连接方式
这个最好实时跟着Oracle的官方文档走，我使用的是``Oracle.ManagerdDataAccess``,把它下载后加入到项目的引用。
Web.config文件的配置如下：
```xml
    <!-- 本地 -->
    <!--<add name="Oracle_DB" connectionString="DATA SOURCE=127.0.0.1/orcl;PASSWORD=bjshadmin;USER ID=ZK" providerName="Oracle.ManagedDataAccess.Client"/>-->
    <!-- 远程 -->
```
配置完成后，我们在后台程序中就可以用下面的方式调用数据库连接和方法：
```csharp
        //创建连接字符串
        String connctionString = ConfigurationManager.ConnectionStrings["Oracle_DB"].ToString();
        OracleConnection conn = new OracleConnection(connctionString); //创建连接
        OracleCommand cmd = conn.CreateCommand();// 创建数据库控制器
        cmd.CommandText = "SELECT * FROM table";//设置commandText
        conn.Open();//开启连接
        /****************查询语句的执行方法 begin******************/
        OracleDataReader reader = cmd.ExecuteReader(); //至此查询结果接收到了reader对象中
        //读取reader对象中的值。
        while (reader.Read())
        {
            StuInfo stu = new StuInfo();
            stu.Ks_zkz = reader[0].ToString();
            stu.Ks_xm  = reader[1].ToString();
            stuList.Add(stu);
        } 
        /*************** 执行Update语句的方法 ******************/ 
        int rows = cmd.ExecuteNonQuery(); //返回受影响条数，删除与增加语句类似

        conn.close();//最后不要忘了close连接  
```

# aspx页面中处理JS的Ajax请求
aspx页面中处理JS的Ajax请求时可以使用``ashx``, _右键->添加->一般处理程序_,一般处理程序可以充当服务器端处理Ajax请求的文件：

JS：
```javascript
        //通过审核的处理
        function passStu_next() {
            var zkzh = $('.kszkz').val();
            alert("标记学号:" + zkzh + "已通过审核");
            $.ajax({
                type: "post",
                url: "passAndNextHandler.ashx",
                data: { "kszkz": zkzh },
                success: function (result) {
                    //跳转到下一个考生的详细内容页面
                    if (result == "") {
                        alert("审核完毕");
                    }
                    else {
                        window.location.href = "DetailCardsInfo.aspx?zkzh=" + result;
                    }
                    
                },
                error: function (result) {
                    alert("error");
                }
            })
        }
```
ashx程序：
```csharp
    public class passAndNextHandler : IHttpHandler, IRequiresSessionState
    {
        //创建连接字符串
        String connctionString = ConfigurationManager.ConnectionStrings["Oracle_DB"].ToString();
        public void ProcessRequest(HttpContext context)
        {
            string czr = HttpContext.Current.Session["UserName"].ToString();
            String nextZkz = null;
            context.Response.ContentType = "text/plain";
            String zkzh = context.Request.Form["kszkz"];
            String sql = "UPDATE ZK.V_BYSQ_BJSH_SKB_KS SET BJSH_JG_SKB = '1',BJSH_SKB_CZR='" + czr + "' WHERE KS_ZKZ= '" + zkzh + "'";
            OracleConnection conn = new OracleConnection(connctionString);
            OracleCommand cmd = conn.CreateCommand();
            conn.Open();
            //向表中插入数据的同时，选择下一个考生
            OracleTransaction transaction = conn.BeginTransaction(IsolationLevel.ReadCommitted);
            cmd.Transaction = transaction; //开始事务
            try
            {
                cmd.CommandText = sql1;
                cmd.ExecuteNonQuery();
                //提交事务
                transaction.Commit(); //这里如果是多条语句，同时提交成功才返回
            }
            catch (Exception ex)
            {
                transaction.Rollback(); //未成功就rollback
                Console.Write(ex.ToString());
            }
            context.Response.Write(nextZkz);
            conn.Close();
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
```

# 访问网络路径中的文件，以及下载文件到本地。
1. 在本地磁盘创建存储路径：
    ```csharp
    String directoryPath = @"D:\Downloadimgs\"+list_zkz[i]+"//"; //定义一个路径变量
    if (!Directory.Exists(directoryPath))//如果路径不存在
    {
        Directory.CreateDirectory(directoryPath);//创建一个路径的文件夹
    }
    ```
2. 网络路径的映射以及文件拷贝
    ```csharp
    //设置源目标路径
    string sourceFile = @"http://127.0.0.1/img//" + reader[0] + "//" + reader[1].ToString().Trim() + "//" + reader[3].ToString().Trim() + ".jpg";
    //设置目的地路径
    string destinationFile = @"D:\Downloadimgs\" + list_zkz[i] + "//" + reader[2] + "_" + reader[1].ToString().Trim() + "_" + reader[0] + ".jpg";
    //FileInfo file = new FileInfo(sourceFile);
    //下面的这种文件拷贝方式，只能用于本地磁盘的文件拷贝，不能用于网络dowmload
    //if (file.Exists)
    //{
    //    // true is overwrite
    //    file.CopyTo(destinationFile, true);
    //}
    WebClient wc = new WebClient(); //创建网络通信实例
    byte[] by = new byte[1024]; //接收数据的数组
    WriteTextLog("beforeCheck:",sourceFile.ToString()+"|"+destinationFile, DateTime.Now);
    WriteTextLog("check:", UrlCheck(sourceFile).ToString(), DateTime.Now);
    if (UrlCheck(sourceFile)) // 这里的Urlcheck是必须的
    {
        WriteTextLog("savefile:", sourceFile.ToString() +"|"+ destinationFile, DateTime.Now);
        FileStream fs = new FileStream(destinationFile, FileMode.Create, FileAccess.Write); //创建文件
        BinaryWriter bw = new BinaryWriter(fs);
        by = wc.DownloadData(sourceFile);
        bw.Write(by, 0, by.Length);
        bw.Close();
        fs.Close(); //关闭数据流
    }    
    ```
3. URL的Check需要判断上面的URL是否是网络地址
    ```csharp
    /**
    * 如果路径中含有http或者https,就进行webrequest的处理
    **/
    public bool UrlCheck(string strUrl)
        {
            if (!strUrl.Contains("http://") && !strUrl.Contains("https://"))
            {
                strUrl = "http://" + strUrl;
            }
            try
            {
                HttpWebRequest myRequest = (HttpWebRequest)WebRequest.Create(strUrl);
                myRequest.Method = "HEAD";
                myRequest.Timeout = 20000;  //超时时间20秒
                HttpWebResponse res = (HttpWebResponse)myRequest.GetResponse();
                Boolean te;
                if (res.StatusCode == HttpStatusCode.OK) te = true;
                else te = false;
                res.Close();
                return te;
                
            }
            catch
            {
                return false;
            }
        }
    ```
4. 上面的日志文件会在项目目录下生成日志文件
    ```csharp
        public static void WriteTextLog(string action, string strMessage, DateTime time)
        {
            string path = AppDomain.CurrentDomain.BaseDirectory + @"System\Log\";
            if (!Directory.Exists(path))
                Directory.CreateDirectory(path);

            string fileFullPath = path + time.ToString("yyyy-MM-dd") + ".System.txt";
            StringBuilder str = new StringBuilder();
            str.Append("Time:    " + time.ToString() + "\r\n");
            str.Append("Action:  " + action + "\r\n");
            str.Append("Message: " + strMessage + "\r\n");
            str.Append("-----------------------------------------------------------\r\n\r\n");
            StreamWriter sw;
            if (!File.Exists(fileFullPath))
            {
                sw = File.CreateText(fileFullPath);
            }
            else
            {
                sw = File.AppendText(fileFullPath);
            }
            sw.WriteLine(str.ToString());
            sw.Close();
        }
    ```