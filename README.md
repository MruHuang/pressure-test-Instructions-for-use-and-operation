[hackmd](https://hackmd.io/@pONMMCWcRMSASTBgLGLkyg/pressure-test-Instructions-for-use-and-operation)

壓測使用與操作說明
======================

Created by mru huang, last modified on Apr 28, 2021

使用工具
====

JMETE
-----

### 前言
  + 這個文件紀錄於電腦上下載、執行 Jmeter 的方式，模擬指定數量的使用者，以錄製之封包於一定時間內執行呼叫，量測流量增加時的 API 回傳時間與穩定性。

### 教學
  + 安裝
    [到官網下載5.3版本](https://downloads.apache.org//jmeter/binaries/apache-jmeter-5.3.zip)

    + 開啟
      解壓縮後執行 /bin/jmeter.bat

  + 範例  [直接open](https://drive.google.com/file/d/1jd3FR9Ylwb7dD4l3RwH8c0K9Ou6L5Dbn/view?usp=sharing)
      

測試參考方面
------

負載測試、壓力測試和效能測試
==============

壓力包含負載，負載包含效能

1.  壓力
    
    1.  壓力測試分為高負載下的長時間（如24小時以上）的穩定性壓力測試和極限負載情況下導致系統崩潰的破壞性壓力測試
        
2.  負載
    
    1.  發現系統可能存在的效能瓶頸、記憶體洩漏、不能實時同步等問題。
        
3.  效能
    
    1.  API的回應時間(指標)
        

測試流程
----

1.  規劃測試情境
    
    1.  要模擬APP使用情境，所以可以要加入CMS的部分一同進行測試(目前還沒加入，有點多，可能也要知道APP有沒有處理)
        
    2.  目前依據get member profile的API數據為其他壓測的參考值(暫定的依據)
        
    3.  應該定義QPS為標準
        
        1.  Queries Per Second，每秒查詢數。每秒能夠響應的查詢次數。
            
        2.  QPS是對一個特定的查詢伺服器在規定時間內所處理流量多少的衡量標準，在網際網路上，作為域名系統伺服器的機器的性能經常用每秒查詢率來衡量。每秒的響應請求數，也即是最大 **吞吐能力**。
            
2.  設定測試規模
    
    1.  同時幾台theard(多人)
        
        1.  一秒直接開高
            
            1.  瞬間流量(併發)
                
    2.  多少時間內達到該人數
        
        1.  這個會用很高的theard
            
            1.  (壓力測試)
                
    3.  測試多久
        
    4.  同時測幾隻API(看流程Test Plan的AP數量)
        
3.  設定方式
    
    1.  123
        
4.  取得壓測資料
    
    1.  open底下的檔案
        

操作
--

在每個API的設定

1.  換環境測試
    
    1.  在JSR223中找String key = '69e6e6b54e76bd1f877fe08b9fb80558';置換成藥測試的環境key
        
    2.  HTTP Header Manager
        
        1.  app-id
            
    3.  Http Requset Defaults
        
        1.  Server Name or IP
            
2.  Test Plan
    
    1.  Run Thread Group
        
        1.  該plan下全部API是否一起跑
            
    2.  各個API
        
        1.  Thread Properties
            
            1.  Number Of Threads
                
                1.  ${\_\_P(threads,)}
                    
                2.  或者自訂亦要開啟的數字
                    
        2.  Ramp-up period
            
            1.  1
                
            2.  自訂義多少時間內開到Number Of Threads的數量
                
        3.  Duration
            
            1.  60
                
            2.  自訂義測試多久時間
                
        4.  Loop Controller
            
            1.  CSV DATA SET CONFIG(測資設定)
                
                1.  fileName(選取測資)
                    

取得測資
====

1.  進入想測試的伺服器API專案的tinker
    
    1.  Loop Controller
        
        1.  CSV DATA SET CONFIG(測資設定)
            
            1.  fileName(選取測資)
                
        2.  進入API的TINKER使用複製貼上該程式片斷
            
            1.  會生成test-data.zip檔案(包含各個測資，可以自行改動程式來生成不同測資)
                
                1.  測試程式
                    
                2.  程式解析
                    ```
                    use Illuminate\\Support\\Facades\\Redis;
                    
                    //會員數量
                    $memberNumber = 300;
                    
                    //會員複製幾次(300 + 300\*2)
                    //同一個會員重複使用的次數
                    $duplicateNumber = 2;
                    
                    //會員資料排序是否為亂序(900的排序為亂數)，且每個檔案的排序也為亂序
                    $random = true;
                    
                    //要生成的檔案，包成zip後會刪除
                    $removedFile = \[
                      'get\_member\_profile.csv',
                      'my\_achievement.csv',
                      'my\_mission\_information.csv',
                      'my\_reward\_redeem\_status.csv',
                      'my\_task\_history.csv',
                      'search\_my\_mission.csv',
                    \];
                    
                    //抓會員(各CSV運用的前置作業)
                    $output = array\_slice(Redis::connection('member')->keys("member.\*.MMRM.tokens"), 0, $memberNumber);
                    $memberIds = \[\];
                    foreach($output as $value) {
                      $memberIds\[\] = explode('.', $value)\[1\];
                    }
                    $redisResponse = Redis::connection('member')->pipeline(function($pipe) use ($memberIds) {
                      foreach ($memberIds as $value){
                      $pipe->lindex("member." . $value  . ".MMRM.tokens", -1);
                      }
                    });  
                    $memberTokens = \[\];
                    foreach($redisResponse as $value) {
                      $memberTokens\[\] = explode('.', $value)\[1\];
                    }
                    
                    $result = \[\];
                    $result = $memberTokens;
                    if ($duplicateNumber > 0) {
                      for ($i = 0; $i < $duplicateNumber; $i++) {
                        $result = array\_merge($result, $memberTokens); 
                      }
                    }
                    $memberTokens = $result;
                    
                    // remove all file
                    exec("rm " . join(" ", $removedFile));
                    
                    
                    //以下為各個CSV獨自的
                    測資生成
                    
                    //-------------------------------------search\_my\_mission---------------------------------
                    //是否要把會員亂數
                    if ($random) shuffle($memberTokens)
                    $filename = "my\_reward\_redeem\_status.csv";
                    $handle = fopen($filename, 'w');
                    
                    //在陣列中塞入該API的資訊(參考Loop Controller底下的JSR223.......中的body)
                    //陣列請轉換成字串(,分隔)，並請在JSR223中使用
                    //class requestParameter {
                    //     def mission\_ids = ${mission\_ids}.split(",");
                    //	boolean full\_info = false;
                    //}
                    //Object request\_parameter = new requestParameter();
                    
                    //會依據CSV DATA SET CONFIG的Variable Nams(comma-de;imited)順序排序
                    $type = \[
                      'task' => '141,74,137,139,143,46,138,96,158,67,48,76,140,135,36,142,47,75,65,66,45,163,54,136',
                      'milestone' => '90,85,51,48,32,41,35,55,70'
                    \];
                    foreach($memberTokens as $memberToken) {
                      $typeKey = array\_rand($type);
                      $typeData = $type\[$typeKey\];
                      fputcsv($handle, \[
                        "'" . $memberToken . "'",
                        "'" . $typeKey . "'",
                        "'" . $typeData . "'",
                        date\_format(now(), 'Y/m/d h:i:s'),
                      \], '~');
                    }
                    fclose($handle);
                    
                    
                    
                    
                    
                    
                    //匯出即刪除暫存檔案
                    // zip all file
                    exec("zip test-data " . join(" ", $removedFile));
                    // remove all file
                    exec("rm " . join(" ", $removedFile));
                    ```
                3.  [範例程式](https://drive.google.com/file/d/1sJxaFVUpVMN3iCtw1w94pIT21JdSs118/view?usp=sharing)
            


1.  Loop Controller
    
    1.  CSV DATA SET CONFIG(測資設定)
        
        1.  fileName(選取測資)
            
    2.  進入API的TINKER使用複製貼上該程式片斷
        
        1.  會生成test-data.zip檔案(包含各個測資，可以自行改動程式來生成不同測資)
            

報表與執行
-----

CMD執行(跑比較快?或者說負仔量會比較高，看電腦效能)，會生成test的報表，請在開啟`jmeter` 的資料夾下執行，並`search_mission_con.jmx` 為open的檔案位子
```
del result.jtl
rd /s /q "test"
jmeter -Jthreads=15 -n -t C:/Users/W541/Desktop/壓測/mru/search\_mission\_con.jmx -l result.jtl
jmeter -g result.jtl -e -o test
```
讀區報表轉換成index.html
```
jmeter -g result.jtl -e -o test
```
