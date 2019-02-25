# MySQL Security and Backup
In this assignment, we have 3 tasks:

## 1. Preventing unauthorized access

Lets assume are several systems which use this database:

- Inventory - which is used to maintain the two tables `products` and `productlines`.
- Bookkeeping which make sure that all orders are payed.
- Human resources which takes care of employees and their offices
- Sales - who creates the orders for the customers
- IT - who maintains this database

Create a database user for each of the four roles, and be restrictive in what the each user can do in the database.

### Solution

- **Inventory**: is given *SELECT*, *INSERT* and *UPDATE* permissions on the `products` and `productlines` table. They need these permissions to maintain product and product line data. They have not been given *DELETE* because it's a bad practice to delete stuff.
    
    ```sql
    /* querry for inventory user */
    CREATE USER 'inventory' IDENTIFIED BY 'password';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.products TO 'inventory'@'%';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.productlines TO 'inventory'@'%'; 
    ```

- **Bookkeeping**: is given *SELECT* on table `customers` and `orders` in case they need to get information about either customers or the orders. But they should not be able to *CREATE*, *UPDATE* or *DELETE* either customers or orders. Then they have been given *SELECT*, *INSERT* and *UPDATE* in the `payments` table and that is because as book keepers, they shoul be able to maintain payments. They should absolutely not be able to *DELETE* payments. 

    ```sql
    /* querry for bookkeeper user */
    CREATE USER 'book-keeper' IDENTIFIED BY 'password';
    GRANT SELECT ON classicmodels.customers TO 'book-keeper'@'%';
    GRANT SELECT ON classicmodels.orders TO 'book-keeper'@'%';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.payments TO 'book-keeper'@'%';
    ```

- **Human Ressources**: have been given *SELECT*, *INSERT* and *DELETE* on `employees` and `offices` table. That is because they should be able to maintain employess and offices. They should not be able to *DELETE* stuff, that should be left for the it admins. 

    ```sql
    /* querry for hr user */
    CREATE USER 'hr' IDENTIFIED BY 'password';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.employees TO 'hr'@'%';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.offices TO 'hr'@'%';
    ```

- **Sales**: is given *SELECT*, *INSERT* and *UPDATE* on table `orders`, `customers`and `orderdetails`. The reason is that if they need to update *status* column on orders, and ofcause they should be able to create new orders and see old. They also need to create new orderdetails and see old and incase they made an error, they should be able to update an orderdetail. Finaly they should be able to see customers and if they make contact with a new customer, they should be able to create a new. They also been given *UPDATE* on customers because it's most likely an sales user a customer would contact if they need to update some of their infomation. 

    ```sql
    /* querry for sales user */
    CREATE USER 'sales' IDENTIFIED BY 'password';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.orders TO 'sales'@'%';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.customers TO 'sales'@'%';
    GRANT SELECT, INSERT, UPDATE ON classicmodels.orderdetails TO 'sales'@'%';
    ```

- **IT**: is being granted all privileges because they need to maintain the entire database. 

    ```sql
    /* querry for it user */
    CREATE USER 'it' IDENTIFIED BY 'password';
    GRANT ALL ON classicmodels.* TO 'it'@'%';
    ```

## 2. Discovering unauthorzed access

Make a number of operations on the database:
- Insert 2 new employees
- Insert 1 new product
- Create 1 new order

To see the log file run this sql command `SELECT event_time, user_host, CONVERT(argument USING utf8 ) AS sqlcommand FROM mysql.general_log;`

![alt text](https://github.com/cph-an178/Sec-and-BK/blob/master/imgs/log.png "Image of the log")

## 3. Recovering from unauthorized access
Create a backup file of the database after the changes in the two previous exercises.

I've choosen to only create an backup of the `classicmodels` database. 

### In a bash terminal:
1. To create an backup file run `exec mysql mysqldump -u root -p classicmodels > dump.sql`
2. To recreate the database:
    - First you must create an database called `classicmodels`
    - Then run `exec mysql mysql -u root --password=<root password> classicmodels < dump.sql`

If you use the docker container write `docker exec -i` instead of `exec`