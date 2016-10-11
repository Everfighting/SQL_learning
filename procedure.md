存储过程

一、使用存储过程的一些好处
1、减少网络流量
2、更加安全
3、代码复用
4、易于维护
5、提高性能

二、存储过程的类型
1、用户定义的
2、临时的
临时过程存储于 tempdb中。 临时过程有两种类型：本地过程和全局过程。
本地临时过程的名称以单个数字符号 (#) 开头；它们仅对当前的用户连接是可见的；当用户关闭连接时被删除。 
全局临时过程的名称以两个数字符号 (##) 开头，创建后对任何用户都是可见的，并且在使用该过程的最后一个会话结束时被删除。
3、系统的
系统过程以前缀 sp_ 开头。

三、创建存储过程
权限：
需要在数据库中有CREATE PROCEDURE权限，对在其中创建过程的架构有ALTER权限。

创建过程：
USE AdventureWorks2012;  
GO  
CREATE PROCEDURE HumanResources.uspGetEmployeesTest2   
    @LastName nvarchar(50),   
    @FirstName nvarchar(50)   
AS   

    SET NOCOUNT ON;  
    SELECT FirstName, LastName, Department  
    FROM HumanResources.vEmployeeDepartmentHistory  
    WHERE FirstName = @FirstName AND LastName = @LastName  
    AND EndDate IS NULL;  
GO  

执行过程：

EXECUTE HumanResources.uspGetEmployeesTest2 N'Ackerman', N'Pilar';  
-- Or  
EXEC HumanResources.uspGetEmployeesTest2 @LastName = N'Ackerman', @FirstName = N'Pilar';  
GO  
-- Or  
EXECUTE HumanResources.uspGetEmployeesTest2 @FirstName = N'Pilar', @LastName = N'Ackerman';  
GO  

四、修改存储过程
创建一个存储过程：
IF OBJECT_ID ( 'Purchasing.uspVendorAllInfo', 'P' ) IS NOT NULL   
    DROP PROCEDURE Purchasing.uspVendorAllInfo;  
GO  
CREATE PROCEDURE Purchasing.uspVendorAllInfo  
WITH EXECUTE AS CALLER  
AS  
    SET NOCOUNT ON;  
    SELECT v.Name AS Vendor, p.Name AS 'Product name',   
      v.CreditRating AS 'Rating',   
      v.ActiveFlag AS Availability  
    FROM Purchasing.Vendor v   
    INNER JOIN Purchasing.ProductVendor pv  
      ON v.BusinessEntityID = pv.BusinessEntityID   
    INNER JOIN Production.Product p  
      ON pv.ProductID = p.ProductID   
    ORDER BY v.Name ASC;  
GO

修改上述存储过程：
ALTER PROCEDURE Purchasing.uspVendorAllInfo  
    @Product varchar(25)   
AS  
    SET NOCOUNT ON;  
    SELECT LEFT(v.Name, 25) AS Vendor, LEFT(p.Name, 25) AS 'Product name',   
    'Rating' = CASE v.CreditRating   
        WHEN 1 THEN 'Superior'  
        WHEN 2 THEN 'Excellent'  
        WHEN 3 THEN 'Above average'  
        WHEN 4 THEN 'Average'  
        WHEN 5 THEN 'Below average'  
        ELSE 'No rating'  
        END  
    , Availability = CASE v.ActiveFlag  
        WHEN 1 THEN 'Yes'  
        ELSE 'No'  
        END  
    FROM Purchasing.Vendor AS v   
    INNER JOIN Purchasing.ProductVendor AS pv  
      ON v.BusinessEntityID = pv.BusinessEntityID   
    INNER JOIN Production.Product AS p   
      ON pv.ProductID = p.ProductID   
    WHERE p.Name LIKE @Product  
    ORDER BY v.Name ASC;  
GO  

执行修改后的存储过程：
EXEC Purchasing.uspVendorAllInfo N'LL Crankarm';  
GO  

五、删除存储过程
权限：需要拥有对该过程所属架构的 ALTER 权限，或对该过程的 CONTROL 权限。

查询存储过程：
SELECT name AS procedure_name   
    ,SCHEMA_NAME(schema_id) AS schema_name  
    ,type_desc  
    ,create_date  
    ,modify_date  
FROM sys.procedures;  

删除存储过程：
DROP PROCEDURE <stored procedure name>;  
GO 

六、执行存储过程：
执行用户定义的存储过程，添加入相应参数值。
USE AdventureWorks2012;  
GO  
EXEC dbo.uspGetEmployeeManagers @BusinessEntityID = 50;  
或
EXEC AdventureWorks2012.dbo.uspGetEmployeeManagers 50;  (只有BusinessEntityID一个参数)
GO  

设置过程自动执行：
USE AdventureWorks2012;  
GO  
EXEC sp_procoption @ProcName = '<procedure name>'   
    , @OptionName = ] 'startup'   
    , @OptionValue = 'on'; 
	
设置过程不自动执行：
USE AdventureWorks2012;  
GO  
EXEC sp_procoption @ProcName = '<procedure name>'
    , @OptionValue = 'off'; 
	
七、授予对存储过程的权限
授予名为 EXECUTE的应用程序角色对存储过程 HumanResources.uspUpdateEmployeeHireInfo 的 Recruiting11权限。  
USE AdventureWorks2012;   
GRANT EXECUTE ON OBJECT::HumanResources.uspUpdateEmployeeHireInfo  
    TO Recruiting11;  
GO  



