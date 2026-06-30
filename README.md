# MAXDOP Hesaplayıcı
SQL Server kurulumunuz için **MAXDOP (Maksimum Paralellik Derecesi)** değerini Microsoft'un resmi önerilerine göre otomatik olarak hesaplayan bir T-SQL sorgusudur.

## MAXDOP Nedir?
**MAXDOP**, SQL Server'ın paralel sorgu planlarında tek bir sorgu için kullanabileceği maksimum işlemci sayısını kontrol eden bir yapılandırma seçeneğidir.
Yanlış yapılandırıldığında aşırı CPU kullanımı,thread çakışmaları ve düşük sorgu performansına sebep olucaktır. Doğru yapılandırıldığında ise optimum paralel sorgu çalışması, dengeli CPU kullanımı ve genel performansta artış yaşanır.
 
## Neden Bu Sorguyu Yazdım?
SQL Server kurulumlarında MAXDOP değerini doğru ayarlamak performans açısından kritik öneme sahip, ancak Microsoft'un önerdiği hesaplama mantığını her seferinde manuel olarak uygulamak hem zaman alıyor hem de hata yapmaya açık.
 
Bu sorguyu, sunucu donanım bilgilerine göre önerilen MAXDOP değerini **otomatik ve tutarlı** bir şekilde hesaplamak için yazdım. Böylece her ortamda aynı mantığı tekrar tekrar elle uygulamak yerine, tek bir sorgu çalıştırarak hızlıca doğru değeri görebiliyorum.
 
## Nasıl Çalışır?
### MAXDOP'un Çalışma Yapısı
 
SQL Server, bir sorguyu çalıştırırken bunu tek bir işlemci üzerinde **seri** olarak ya da birden fazla işlemci kullanarak **paralel** olarak çalıştırabilir. MAXDOP, bu paralel çalışmada kullanılacak maksimum işlemci sayısını belirler.
 
- **MAXDOP = 0** → SQL Server tüm mevcut CPU çekirdeklerini kullanabilir. Yoğun sistemlerde sorun yaşanabilir. (varsayılan)
- **MAXDOP = 1** → Paralellik tamamen kapatılır. İşlemler tek bir CPU çekirdeği üzerinden sırayla çalıştırılır.
- **MAXDOP = N** → Sorgu başına maksimum N işlemci kadar kullanılır.
 
Yanlış yapılandırılmış MAXDOP değeri, özellikle çok işlemcili ve çok NUMA node'lu sunucularda ciddi performans sorunlarına yol açabilir.
 
### Sorgunun Yaptığı
Bu sorgu **sys.dm_os_sys_info** ve **sys.configurations** sistem görünümlerinden sunucuya ait donanım bilgilerini okuyarak mevcut MAXDOP değerini ve önerilen değeri hesaplar.

**Hesaplama mantığı 4 senaryoya göre çalışır:**
 
| Sunucu Konfigürasyonu 						   | İşlemci Sayısı                         | Önerilen MAXDOP Değeri       									                  |
|--------------------------------------|----------------------------------------|-----------------------------------------------------------------|
| Single Numa Node       						   | ≤ 8 mantıksal CPU       				        | CPU sayısı kadar veya daha altında olmalıdır.           		    |
| Single Numa Node       						   | > 8 mantıksal CPU       				        | 8   olmalıdır                         						              |
| Multiple Numa Node      						 | NUMA başına ≤ 16 CPU  					        | NUMA başına CPU sayısı veya daha altında olmalıdır.         	  |
| Multiple Numa Node         					 | NUMA başına > 16 CPU  					        | NUMA başına CPU / 2 kadar olmalıdır. (maxsimum 16 olabilir) 	  |
 
**NOT:** Sorgu aynı zamanda mevcut 'value_in_use' değerini de kontrol eder. Eğer mevcut değer zaten önerilen aralıktaysa, gereksiz yere değişiklik önermeyecektir.
  
## Uyumluluk
SQL Server 2016 (13.x) ve sonraki sürümlerinde sorunsuz çalıştırılabilir.
 
## Örnek Çıktı
| Server Name        | (No column name) |	(No column name)                                            |	Current_MAXDOP_Value|	recommended_maxdop_max_value|
|--------------------|------------------|-------------------------------------------------------------|---------------------|-----------------------------|
|örnek1	             |CPU	              |4 logical processors, 2 physical cores, 1 NUMA node(s)	      |4	                  |4                            |
|örnek2	             |CPU	              |8 logical processors, 1 physical cores, 1 NUMA node(s)	      |0	                  |8                            |
|örnek3              |CPU               |12 logical processors, 1 physical cores, 2 NUMA node(s)	    |0	                  |6                            |

## Kaynaklar
https://learn.microsoft.com/tr-tr/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option
https://www.sqlshack.com/importance-of-sql-server-max-degree-of-parallelism/

 
