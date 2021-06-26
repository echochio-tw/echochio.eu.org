---
layout: post
title: MYSQL a foreign key constraint fails
date: 2021-06-22
tags: mysql foreign key DELETE
---

MYSQL 有外鍵無法刪除如何處理?

```
SQLSTATE[23000]: Integrity constraint violation: 1451 Cannot delete or update a parent row: a foreign key constraint fails 
(`Customer_info`.`amountinfo`, CONSTRAINT `amountinfo_ibfk_2` 
FOREIGN KEY (`customer_id`) REFERENCES `customer` (`customer_id`) 
ON DELETE NO ACTION ON UPDATE NO ACTION)
```

要確定刪除沒影響就可刪除, 不然就會變成無主的外鍵
```
SET FOREIGN_KEY_CHECKS=0; -- to disable them
DELETE FROM customer WHERE customer_account = "test_user";
SET FOREIGN_KEY_CHE
```
