.. code:: ipython3

    import sqlite3
    
    conn = sqlite3.connect('Assesment_Pandas_Updates.db')
    
    cursor = conn.cursor()

.. code:: ipython3

    cursor.execute('''CREATE TABLE IF NOT EXISTS Customer_Table (
                       CUSTOMER_ID TEXT PRIMARY KEY,
                       Age INTEGER NOT NULL
                       )''')




.. parsed-literal::

    <sqlite3.Cursor at 0x7ff301dc69c0>



.. code:: ipython3

    conn.execute("INSERT INTO Customer_Table (Customer_ID , Age) \
                  VALUES ('C1001', 17)");
    
    conn.execute("INSERT INTO Customer_Table (Customer_ID , Age) \
                  VALUES ('C2002', 19)");
    
    conn.execute("INSERT INTO Customer_Table (Customer_ID , Age) \
                  VALUES ('C3001' , 20)");
    
    cursor.execute("SELECT * FROM Customer_Table")
    
    rows = cursor.fetchall()

.. code:: ipython3

    cursor.execute('''CREATE TABLE IF NOT EXISTS Sales_Table (
                       Sales_ID INTEGER PRIMARY KEY,
                       CUSTOMER_ID TEXT NOT NULL
                          )''')
    
    conn.execute("INSERT INTO Sales_Table (Sales_ID , CUSTOMER_ID) \
                  VALUES (11,'C1001' )");
    
    conn.execute("INSERT INTO Sales_Table (Sales_ID , CUSTOMER_ID) \
                  VALUES (12,'C2002')");
    
    
    cursor.execute("SELECT * FROM Sales_Table")
    
    rows = cursor.fetchall()


.. code:: ipython3

    cursor.execute('''CREATE TABLE IF NOT EXISTS Orders_Table (
                       Order_ID TEXT PRIMARY KEY,
                       Sales_ID Integer NOT NULL,
                       Item_Id TEXT NOT NULL,
                       Quantity Integer
                       )''')
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord01', 10 , 'it01' , 2)");
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord02', 10 , 'it02' , 3)");
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord03', 10 , 'it03' , Null)");
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord04', 11 , 'it01' , 1)");
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord05', 11 , 'it02' , Null)");
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord06', 11 , 'it03' , Null)");
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord07', 12 , 'it01' , 1)");
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord08', 12 , 'it02' , 2)");
    
    
    conn.execute("INSERT INTO Orders_Table (Order_ID , Sales_ID , Item_ID , Quantity) \
                  VALUES ('Ord09', 12 , 'it03' , Null)");
    
    
    cursor.execute("SELECT * FROM Orders_Table")
    
    rows = cursor.fetchall()

.. code:: ipython3

    query = """
        CREATE TABLE customer_sales_data_new AS
        SELECT Sales_table.Sales_Id , Sales_table.Customer_Id , Customer_Table.Age
        FROM Sales_table
        INNER JOIN Customer_Table ON Sales_table.customer_id = Customer_Table.customer_id
    """
    
    cursor.execute(query)
    
    cursor.execute("SELECT * FROM customer_sales_data_new")
    
    rows = cursor.fetchall()
    


.. code:: ipython3

    import pandas as pd
    
    # Using Pandas to read data from the SQL table into a DataFrame
    df = pd.read_sql_query("SELECT * FROM customer_sales_data_new", conn)

.. code:: ipython3

    print(df)



.. parsed-literal::

       Sales_ID CUSTOMER_ID  Age
    0        11       C1001   17
    1        12       C2002   19


.. code:: ipython3

    filtered_df = df.query('Age > 18')

.. code:: ipython3

    print(filtered_df)



.. parsed-literal::

       Sales_ID CUSTOMER_ID  Age
    1        12       C2002   19


.. code:: ipython3

    orders_df = pd.read_sql_query("SELECT * FROM Orders_Table" , conn)

.. code:: ipython3

    print(orders_df)


.. parsed-literal::

      Order_ID  Sales_ID Item_Id  Quantity
    0    Ord01        10    it01       2.0
    1    Ord02        10    it02       3.0
    2    Ord03        10    it03       NaN
    3    Ord04        11    it01       1.0
    4    Ord05        11    it02       NaN
    5    Ord06        11    it03       NaN
    6    Ord07        12    it01       1.0
    7    Ord08        12    it02       2.0
    8    Ord09        12    it03       NaN


.. code:: ipython3

    merged_df = pd.merge(filtered_df, orders_df, on='Sales_ID', how='inner')
    
    print(merged_df)


.. parsed-literal::

       Sales_ID CUSTOMER_ID  Age Order_ID Item_Id  Quantity
    0        12       C2002   19    Ord07    it01       1.0
    1        12       C2002   19    Ord08    it02       2.0
    2        12       C2002   19    Ord09    it03       NaN


.. code:: ipython3

    merged_df_new = merged_df.dropna(subset=['Quantity'])

.. code:: ipython3

    # The final output
    
    print(merged_df_new)


.. parsed-literal::

       Sales_ID CUSTOMER_ID  Age Order_ID Item_Id  Quantity
    0        12       C2002   19    Ord07    it01       1.0
    1        12       C2002   19    Ord08    it02       2.0


