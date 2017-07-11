# 目标

* 爬取Instagram不同明星的粉丝数据.
* 爬取2万粉丝数据，其中包括粉丝账号名称，粉丝名称和粉丝头像数据.
* 并且存储爬取得数据到csv文件中.
* 打包成一个小程序可以直接被非技术人员使用.

## instagram网站分析与任务拆分

1. Instagram登录界面.  
   1. 登录界面无复杂验的证码，初步考虑直接模拟用户cookie登录的方式.

2. 目标爬取页面.

   1. 用户粉丝页面数据动态加载，并且提交的参数是通过加密的方式提交.

## 最后实现方式

```
package us.codecraft.webmagic.downloader.selenium;

import com.csvreader.CsvWriter;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Element;
import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.interactions.Actions;
import org.openqa.selenium.logging.LogType;
import org.openqa.selenium.logging.LoggingPreferences;
import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;
import us.codecraft.xsoup.XElements;
import us.codecraft.xsoup.XPathEvaluator;
import us.codecraft.xsoup.Xsoup;

import javax.xml.xpath.XPath;
import java.io.File;
import java.io.IOException;
import java.nio.charset.Charset;
import java.util.*;
import java.util.logging.Level;

/**
 * Created with IntelliJ IDEA
 * Project: webmagic-parent
 * User: chenhuiyang
 * Date: 2017/7/7
 * TIME: 上午10:00
 */
public class YZBSeleniumProcessorMain {


    //用户登录账号
    public static String USER_NAME = "账户名称";

    //用户登录密码
    public static String PASSWORD = "密码";

    //明星名称
    public static String STAR_NAME = "justinbieber";

    //chromedriver驱动地址
    public static String WEBDRIVER_CHROME_DRIVER = "/Users/chenhuiyang/Downloads/chromedriver";

    public static String INSTAGRAM_LOGIN_PAGE_URL = "https://www.instagram.com/accounts/login/";

    public static  int FENSI_SIZE = 20000;

    public static  int PAGE_COUNT = 2000;

    public static void main(String[] args) throws InterruptedException {

        Scanner sc=new Scanner(System.in);
        System.out.println("***************FireBird爬虫程序: 开始*************************");
        System.out.println("请输入chromdriver文件地址:[/Users/chenhuiyang/Downloads/chromedriver]");
        String web_driver_address = sc.next();
        if(web_driver_address == null || web_driver_address.trim().equals("")){
            System.out.println("chromdriver文件地址输入错误，程序将会退出");
            return;
        }
        WEBDRIVER_CHROME_DRIVER = web_driver_address;
        System.out.println("您输入的chromdriver文件地址是:" + WEBDRIVER_CHROME_DRIVER);


        System.out.println("请输入要爬取的明星名称:[justinbieber]");
        String star_name = sc.next();
        if(star_name == null || star_name.trim().equals("")){
            System.out.println("明星名称输入错误，程序将会退出");
            return;
        }
        STAR_NAME = star_name;
        System.out.println("您输入的要爬取的明星名称是:" + STAR_NAME);


        System.out.println("请输入要爬取的粉丝的数据条数:["+FENSI_SIZE+"]");
        int fensi_size = sc.nextInt();
        if(fensi_size <= 0){
            System.out.println("要爬取的粉丝的数据条数输入错误，程序将会退出");
            return;
        }
        PAGE_COUNT = (fensi_size%10 == 0 ? ((fensi_size/10 )+1) : fensi_size/10);
        if(PAGE_COUNT > 1000){
            System.out.println("温馨提示:请求的粉丝数据量比较大,耗费事件会很久!");
        }
        System.out.println("您输入爬取的粉丝的数据条数:" + FENSI_SIZE  + " : " + " 将会依次请求" + PAGE_COUNT + "次!");

        System.out.println("请输入要账号名称:["+USER_NAME+"]");
        String user_name = sc.next();
        if(user_name == null || user_name.trim().equals("")){
            System.out.println("登录账号输入错误，程序将会退出");
            return;
        }
        USER_NAME = user_name;
        System.out.println("您输入要账号名称为:" + USER_NAME);


        System.out.println("请输入要账号密码:["+PASSWORD+"]");
        String password = sc.next();
        if(password == null || password.trim().equals("")){
            System.out.println("登录账号密码输入错误，程序将会退出");
            return;
        }
        PASSWORD = password;
        System.out.println("您输入要账号密码为:" + PASSWORD);



        System.out.println();
        System.out.println("***********************参数输入完毕!*********************************");
        System.out.println(USER_NAME + ":" + PASSWORD + ":" + STAR_NAME + ":" + FENSI_SIZE + ":" + WEBDRIVER_CHROME_DRIVER);
        System.out.println();

        //设置chromedriver的地址
        System.getProperties().setProperty("webdriver.chrome.driver", WEBDRIVER_CHROME_DRIVER);

        //不现实任何图片的配置 。 设置无效果 暂时先留着
        Map<String, Object> contentSettings = new HashMap<String, Object>();
        contentSettings.put("images", 2);

        Map<String, Object> preferences = new HashMap<String, Object>();
        preferences.put("profile.default_content_settings", contentSettings);




        DesiredCapabilities caps = DesiredCapabilities.chrome();

        LoggingPreferences logPrefs = new LoggingPreferences();
        logPrefs.enable(LogType.PERFORMANCE, Level.ALL);
        caps.setCapability(CapabilityType.LOGGING_PREFS, logPrefs);
        caps.setCapability("chrome.prefs", preferences);
        caps.setCapability("chrome.switches", Arrays.asList("--user-data-dir=/Users/chenhuiyang/temp/chrome"));
        WebDriver webDriver = new ChromeDriver(caps);
        webDriver.manage().window().maximize();
        //获取instagram登录页面
        webDriver.get(INSTAGRAM_LOGIN_PAGE_URL);

        //用户名称自动输入
        webDriver.findElement(By.name("username")).clear();
        webDriver.findElement(By.name("username")).sendKeys(USER_NAME);

        //休眠两秒
        Thread.sleep(2000l);

        //用户密码的自动输入
        webDriver.findElement(By.name("password")).clear();
        webDriver.findElement(By.name("password")).sendKeys(PASSWORD);

        //休眠两秒
        Thread.sleep(2000l);

        //模拟登录按钮的点击事件
        webDriver.findElement(By.className("_rmr7s")).click();

        //休眠两秒
        Thread.sleep(2000l);

        //获取个人页面
        webDriver.get("https://www.instagram.com/"+STAR_NAME+"/");

        //休眠三秒，等待justinbieber的网页正常加载完成
        Thread.sleep(3000l);

        //获取正在关注的用户列表
        System.out.println(webDriver.findElements(By.className("_s53mj")).get(1).getText());

        //模拟粉丝列表弹窗谈起的功能
        JavascriptExecutor javascriptExecutor = (JavascriptExecutor) webDriver;
        javascriptExecutor.executeScript("arguments[0].click();",webDriver.findElements(By.className("_s53mj")).get(1));

        //休眠4秒，等待用户的数据加载完成
        Thread.sleep(4000l);

        String filePath = "/Users/chenhuiyang";

        String usrHome = System.getProperty("user.home");

        String downloads = usrHome + "/Downloads/";

        String fileName = UUID.randomUUID().toString();

        String cvsFilePath = downloads + fileName + ".csv";

        File cvsFilePathFile  = new File(cvsFilePath);
        if(!cvsFilePathFile.exists()){
            try {
                cvsFilePathFile.createNewFile();
                System.out.println("构建文件成功......");
            } catch (IOException e) {
                e.printStackTrace();
            }
        }


        CsvWriter csvWriter = new CsvWriter(cvsFilePath,',', Charset.forName("UTF-8"));

        System.out.println(usrHome);


        List<WebElement> cx1ua = webDriver.findElements(By.className("_6jvgy"));
        //webDriver.findElements(By.xpath("//"));
        List<String[]> datas = new ArrayList<String[]>();
        for(int i = 0 ; i < 20 ; i ++){
            try{
                WebElement webElement = cx1ua.get(i);
                String imgSrc = webElement.findElement(By.className("_a012k")).getAttribute("src");
                String userName = webElement.findElement(By.className("_4zhc5")).getAttribute("title");
                String realdName = webElement.findElement(By.className("_2uju6")).getText();

                System.out.println("抓取第[" + i + "]个" + "粉丝数据: " + userName + " : " + realdName + " :" + imgSrc);
                csvWriter.writeRecord(new String[]{userName , realdName , imgSrc});
                csvWriter.flush();
                datas.add(new String[]{userName , realdName , imgSrc});
            }catch (Exception e){
                e.printStackTrace();
                continue;
            }
        }

        javascriptExecutor.executeScript("arguments[0].scrollIntoView();",cx1ua.get(19));


        Thread.sleep(4000);
        //模拟鼠标点击操作
        //Actions actions = new Actions(webDriver);




        XPathEvaluator compile = Xsoup.compile("//div[@class='_6jvgy']");
        XPathEvaluator compile1 = Xsoup.compile("//img[@class='_a012k']");
        XPathEvaluator compile2 = Xsoup.compile("//a[@class='_4zhc5']");
        XPathEvaluator compile3 = Xsoup.compile("//div[@class='_2uju6']");

        for (int i = 0 ; i < PAGE_COUNT; i ++){
            System.out.println("**********************开始第"+i +"次抓取数据*************************");
            //cx1ua = webDriver.findElements(By.className("_6jvgy"));
            long time =System.currentTimeMillis();

            XElements evaluate = compile.evaluate(Jsoup.parse(webDriver.getPageSource()));

            System.out.println("第" + i + "解析时间:"+(System.currentTimeMillis()-time));

            int size = evaluate.getElements().size();
            System.out.println(size + " 获取现在的数据大小 ");
            int requestSize = size - 20;
            int realRequestPageCount = ((requestSize % 10 != 0) ? (requestSize /10) + 1 : requestSize/10);
            if(realRequestPageCount-1 != i){
                i = realRequestPageCount - 1;
                continue;
            }
            System.out.println(" realRequestPageCount : " + realRequestPageCount);
            for(int k = 20 + i*10; k < (i+1) * 10 + 20 && k < size ; k ++){
                    try{
                        Element element = evaluate.getElements().get(k);
                        XElements evaluate1 = compile1.evaluate(element);
                        String imgSrc = evaluate1.getElements().get(0).attr("src");

                        XElements evaluate2 = compile2.evaluate(element);
                        String userName = evaluate2.getElements().get(0).attr("title");

                        XElements evaluate3 = compile3.evaluate(element);
                        String realdName = evaluate3.getElements().get(0).text();

                        System.out.println("抓取第[" + k + "]个" + "粉丝数据: " + userName + " : " + realdName + " :" + imgSrc);
                        time = System.currentTimeMillis();
                        csvWriter.writeRecord(new String[]{userName , realdName , imgSrc});
                        csvWriter.flush();
                        System.out.println("写文件时间:"+(System.currentTimeMillis()-time));

                        //System.out.println("抓取第[" + k + "]个" + "粉丝数据: " + userName + " : " + realdName + " :" + imgSrc);
                        /*
                        evaluate1.
                        String imgSrc = webElement.findElement(By.className("_a012k")).getAttribute("src");
                        String userName = webElement.findElement(By.className("_4zhc5")).getAttribute("title");
                        String realdName = webElement.findElement(By.className("_2uju6")).getText();
                        System.out.println("抓取第[" + k + "]个" + "粉丝数据: " + userName + " : " + realdName + " :" + imgSrc);
                        csvWriter.writeRecord(new String[]{userName , realdName , imgSrc});
                        csvWriter.flush();
                        */
                        //datas.add(new String[]{userName,realdName,imgSrc});
                    }catch (Exception e){
                        e.printStackTrace();
                        continue;
                    }
            }
            try{
                //actions.moveToElement(cx1ua.get(size - 1)).perform();
                //System.out.println( "sdfsdfsdf " + webDriver.findElements(By.xpath("//li[@class='_cx1ua']")).size());
                javascriptExecutor.executeScript("arguments[0].scrollIntoView();",webDriver.findElement(By.xpath("//li[@class='_cx1ua']["+(size-1)+"]")));
                Thread.sleep(2000);
            }catch (Exception e){
                e.printStackTrace();
                //cx1ua = webDriver.findElements(By.className("_6jvgy"));
                //size = cx1ua.size();
                javascriptExecutor.executeScript("arguments[0].scrollIntoView();",webDriver.findElement(By.xpath("//li[@class='_cx1ua']["+(size-1)+"]")));
                Thread.sleep(2000);
                //actions.moveToElement(cx1ua.get(size - 1)).perform();
            }


            System.out.println("**********************结束第"+i +"次抓取数据*************************" + "解析时间:"+(System.currentTimeMillis()-time));
        }



        csvWriter.flush();
        csvWriter.close();
        System.out.println("抓取数据成功, 3s之后退出 ");

        //休眠三秒关闭 浏览器页面
        Thread.sleep(3000l);

        System.out.println("关闭页面!");
        //
        webDriver.close();
        System.exit(0);

    }

}
```

## 不足之处

后期发现在爬取数据到5000条左右的时候，会出现chrome卡死的情况，现在初步怀疑是因为dom节点过多导致 ,。目前未找到优化解决方案，如果有哪位大神解决可以联系351361231@qq.com。



