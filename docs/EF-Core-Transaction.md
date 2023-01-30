# EF-Core-Transaction

處理DB Roll back

* 透過 DbContext.DataBase.BeginTransaction() 可取得 IDbContextTransaction，可將多次 SaveChanges() 包成一個 Transaction，由程式決定何時 Commit 或 Rollback。

若在 Commit 前發生例外，交易將會被 roll back

```
public void toTest1()
        {
            using (var tran = dbCtx.Database.BeginTransaction())
            { 
                //....
                dbCtx.SaveChanges();
                //....
                dbCtx.SaveChanges();
                tran.Commit();
            }
        }
```



* 透過 System.Transactions.TransactionScope()

若在 complete 前發生例外，包含在 scope 內的交易將會被 roll back

```
public void toTest2()
        {
            using (var scope = new System.Transactions.TransactionScope())
            { 
                //....
                dbCtx.SaveChanges();
                //....
                //Call Api
                
                //....
                dbCtx.SaveChanges();
                scope.Complete();
            }
        }
        
```
