# TDR Migration

## 1. Task
You have been assigned the following task, please make sure your code is tested and well documented in the read-me file of this Github repository.

####Introduction

A customer is requesting you to push their daily TDR (Transaction Detailed Records) to their mediation server so they can generate graphs and Advanced reports, They request to send them the data in CSV format via Secure FTP. The TDR is pulled from three tables in the MySQL Database.

SMS Detailed Records table: smpp_messages
image-20210624-024000

####Useful PHP sample:
```
$sent_sql = "";
if($sent == 1 || $sent == 0 ) {
$sent_sql = " AND s.sent=$sent ";
}

$customer_sql = "";
if($i_customer > 0 ) {
$customer_sql = " AND s.i_customer = $i_customer ";
}

$vendor_sql = "";
if($i_vendor > 0 ) {
$vendor_sql = " AND s.i_vendor = $i_vendor ";
}

// Connection refers to route column

$connection_sql = "";
if($i_connection > 0 ) {
$connection_sql = " AND s.i_connection = $i_connection ";
}

$qry = "SELECT COUNT(*) FROM smpp_messages s WHERE s.ts between '$date_from' AND '$date_to' $sent_sql $customer_sql $vendor_sql  $connection_sql  $where_prefix ";

        $res = mysql_query($qry, $slave_db) or die(mysql_error($slave_db).$qry);
        $row = mysql_fetch_row($res);
        $total_rows = $row[0];
        if($total_rows > 0 ) {
                $links = ceil($total_rows/$pager)-1;
                $smarty->assign('links', range(0,$links) );
        }
        $smarty->assign('maxvalue', ($links*$pager));
        $smarty->assign('total_rows', $total_rows );

}

$qry = "SELECT s.*, v.login as vendor_name, a.name as account_name,
con.name as connection_name, con.anonymous, cus.login as customer_name
FROM smpp_messages s
JOIN vendors v ON (v.i_vendor = s.i_vendor)
JOIN customers cus ON (cus.i_customer = s.i_customer)
JOIN accounts a ON (a.i_account = s.i_account)
JOIN connections con ON (con.i_connection = s.i_connection)
WHERE s.ts between '$date_from' AND '$date_to'  $where_prefix
$sent_sql $customer_sql $vendor_sql  $connection_sql
ORDER BY s.ts DESC $limits";
Voice Call Detailed Records table: cdrs
Screen Shot 2021-06-29 at 2 29 54 PM

To help you develop faster here is sample queries/php code that could be useful:

$where_buyer = " ";
if(strlen($buyer) > 0 ) {
$qry = "SELECT i_customer FROM customers where login like '$buyer'";
$res = mysql_query($qry, $slave_db) or die(mysql_error($slave_db));
$row = mysql_fetch_row($res);
$where_buyer = " AND c.i_customer = '".$row[0]."' ";
}

$where_seller = " ";
if(strlen($seller) > 0 ) {
$qry = "SELECT i_vendor FROM vendors where login like '$seller'";
$res = mysql_query($qry, $slave_db) or die(mysql_error($slave_db));
$row = mysql_fetch_row($res);
$where_seller = " AND c.i_vendor = '".$row[0]."' ";
}

$where_route = " ";
if(strlen($route) > 0 ) {
$qry = "SELECT i_connection FROM connections c where name like '$route' $where_seller ";
$res = mysql_query($qry, $slave_db) or die(mysql_error($slave_db));
$row = mysql_fetch_row($res);
$where_route = " AND c.i_connection = '".$row[0]."' ";
}


/// good means successfull calls, bad referring to un-sucessfull calls. else referring to all (bad and good) CDRs

if($show == "good") {
$qry = "SELECT COUNT(*) FROM cdrs c WHERE c.setup_time between '$date_from' AND '$date_to' $where_prefix $where_buyer $where_seller $where_route ";
} elseif ($show == "bad") {
$qry = "SELECT COUNT(*) FROM cdrs_zero c WHERE c.setup_time between '$date_from' AND '$date_to' $where_prefix $where_buyer $where_seller $where_route";
} else {
$qry = "SELECT  (SELECT COUNT(*) FROM cdrs_zero c WHERE  c.setup_time between '$date_from' AND '$date_to' $where_prefix $where_buyer $where_seller $where_route) +
(SELECT COUNT(*) FROM cdrs c WHERE c.setup_time between '$date_from' AND '$date_to' $where_prefix $where_buyer $where_seller $where_route) ";
}
``````
HLR Lookups table_name: hlr_lookup_log
For HLR , Currently data is stored in DB as JSON object, you must parse and export to CSV in a similar structure as the above.

####Notes:

Development Environment = Ubuntu 16.04 LTS, PHP7, MySQL 5.7
The customer may ask for other tables in the future to be pushed to them in the same way, you must consider this when building your solution.
Daily there are about 500k-1M records and forecasted to reach 3-5 Million records a day.
Customer expects a log file to trace the progress, history of transfers.
Customer wants to have control over the retention policy of transferred data
Customer wants to have exception handling, if a file fails to transfer in the first attempt, please implement an alerting and retry strategy.
Hint: you can make this controlable from web or a new DB table.

We expect you to write the complete tech documentation for setting up the new script including cron-tab configuration steps.