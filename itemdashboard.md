# Item Dashboard report

## How to create the item dashboard report

The SQL for this report is kind of a lot:

<mark>(Warning - if you copy and paste this report directly into your system - it will not work until you create the accessory reports and edit this SQL with the correct report numbers)</mark>

~~~SQL

SELECT
  Concat_Ws('<br />',
  '<h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog</h3>',
  Concat('Item homebranch: ', items.homebranch),
  Concat('Current branch: ', items.holdingbranch),
  Concat('Permanent shelving location: ', items.permanent_location),
  Concat('Current shelving location: ', items.location),
  Concat('Item type: ', items.itype),
  Concat('Collection code: ', ccodes.lib),
  Concat('Call#: ', items.itemcallnumber),
  Concat('Author: ', biblio.author),
  Concat('Title: ',
  Concat_Ws(' ',
    '<span style="text-transform: uppercase">',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    '</span>')
  ),
  Concat('Item barcode: ', Upper(items.barcode)),
  Concat('<br />Public notes: ', items.itemnotes),
  Concat('Non-public notes: ', items.itemnotes_nonpublic),
  Concat('<br />Total circulation: ', (Sum((Coalesce(items.issues, 0)) + (Coalesce(items.renewals, 0))))),
  Concat('(', items.issues, ' checkouts + ', items.renewals, ' renewals)'),
  Concat('<br />Date added: ', items.dateaccessioned),
  Concat('Last borrowed: ', items.datelastborrowed),
  Concat('Last seen: ', items.datelastseen),
  Concat('Item record last modified: ', items.timestamp),
  Concat('<br />Due date: ', If(issuesi.date_due IS NULL, "-", Date_Format(issuesi.date_due, "%Y.%m.%d"))),
  Concat("Not for loan status: ", If(items.notforloan = 0, "-", If(items.notforloan IS NULL, "-", nfl.lib))),
  Concat("Damaged status: ", If(items.damaged = 0, "-", If(items.damaged IS NULL, "-", damagedi.lib))),
  Concat(
    "Lost status: ",
    If(
      items.itemlost = 0,
      "-",
      If(items.itemlost IS NULL, "-", Concat(losti.lib, " on ", items.itemlost_on))
    )
  ),
  Concat(
    "Withdrawn status: ",
    If(
      items.withdrawn = 0,
      "-",
      If(items.withdrawn IS NULL, "- ", Concat(withdrawni.lib, " on ", items.withdrawn_on))
    )
  ),
  Concat(
    "<br /> In transit from ",
    If(
      transfersi.frombranch IS NULL,
      "-",
      Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
    )
  ),
  Concat(
    "<br />Link to borrower: ",
    If(
      issuesi.date_due IS NULL,
      "-",
      Concat(
        "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
        issuesi.borrowernumber,
        "' target='_blank'>go to the borrower's account</a>"
      )
  )
  ),
  Concat(
    "Link to title: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the bibliographic record</a>"
    )
  ),
  Concat(
    "Link to item: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
      items.itemnumber,
      "&biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the item record</a>"
    )
  ),
  Concat(
    "Item circ history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
      items.barcode,
      "' target='_blank'>see item circ history</a>"
    )
  ),
  Concat(
    "Item action log history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
      items.itemnumber,
      "' target='_blank'>see action log history</a>"
    )
  ),     
  Concat(
    "Item in transit history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
      items.barcode,
      "' target='_blank'>see item transit history</a>"
    )
  ),
  Concat(
    "Request history on this title: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      biblio.biblionumber,
      "&sql_params=%25' target='_blank'>see title's request history</a>"
    )
  ),
  Concat(
    "Request history on this item: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      items.barcode,
      "' target='_blank'>see item's request history</a>"
    )
  ),
  '<br /><h3 style="color: white; background-color: #829356; text-align: center;">This item is currently in the catalog<br />it has not been deleted</h3>'
  ) AS INFO
FROM
  items
JOIN biblio
  ON items.biblionumber = biblio.biblionumber
JOIN biblio_metadata
  ON biblio_metadata.biblionumber = biblio.biblionumber AND
     items.biblionumber = biblio_metadata.biblionumber
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE'
    ) ccodes
  ON items.ccode = ccodes.authorised_value
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN'
    ) nfl
  ON items.notforloan = nfl.authorised_value
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED'
    ) damagedi
  ON items.damaged = damagedi.authorised_value
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST'
    ) losti
  ON items.itemlost = losti.authorised_value
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN'
    ) withdrawni
  ON items.withdrawn = withdrawni.authorised_value
LEFT JOIN (
    SELECT
      branchtransfers.itemnumber,
      branchtransfers.frombranch,
      branchtransfers.datesent,
      branchtransfers.tobranch,
      branchtransfers.datearrived
    FROM
      branchtransfers
    WHERE
      branchtransfers.datearrived IS NULL
    ) transfersi
  ON items.itemnumber = transfersi.itemnumber
LEFT JOIN (
    SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues
    ) issuesi
  ON items.itemnumber = issuesi.itemnumber
WHERE
  items.barcode LIKE Concat("%", <<Enter barcode number>>, "%")
GROUP BY
  items.itemnumber
UNION
SELECT
  Concat_Ws('<br />',
  '<h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item has been deleted</h2>',
  Concat(
    'At the time of its deletion on:  <ins><strong>',
    deleteditems.timestamp,
    "<br /></strong></ins> this item's information was as follows:<br />"
  ),
  Concat('Item homebranch: ', deleteditems.homebranch),
  Concat('Current branch: ', deleteditems.holdingbranch),
  Concat('Permanent shelving location: ', deleteditems.permanent_location),
  Concat('Current shelving location: ', deleteditems.location),
  Concat('Item type: ', deleteditems.itype),
  Concat('Collection code: ', ccodes.lib),
  Concat('Call#: ', deleteditems.itemcallnumber),
  Concat('Author: ', Coalesce(biblio.author, deletedbiblio.author)),
  Concat('Title: ', Coalesce(biblio.title, deletedbiblio.title)),
  Concat('Item barcode: ', deleteditems.barcode),
  Concat('Replacement price: ', deleteditems.replacementprice),
  Concat('Item id number: ', deleteditems.itemnumber),
  Concat(
    "<br />Damaged status: ",
    If(
      deleteditems.damaged = 0,
      "-",
      If(deleteditems.damaged IS NULL, "-", damagedi.lib)
    )
  ),
  Concat(
    "Lost status: ",
    If(
      deleteditems.itemlost = 0,
      "-",
      If(deleteditems.itemlost IS NULL, "-", Concat(losti.lib, " on ", deleteditems.itemlost_on))
    )
  ),
  Concat(
    "Withdrawn status: ",
    If(
      deleteditems.withdrawn = 0,
      "-",
      If(
        deleteditems.withdrawn IS NULL,
        "- ",
        Concat(deletedwithdrawni.lib, " on ", deleteditems.withdrawn_on)
      )
    )
  ),
  If(
    biblio.biblionumber IS NULL,
    "<br />-- Bibliographic record has been deleted --",
    Concat(
      "<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>Go to the bibliographic record</a>"
    )
  ),
  Concat(
    "<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=",
    Replace(
      Replace(
        Replace(
          Replace(
            Replace(
              Replace(
                Replace(
                  deleteditems.barcode,
                  Char(43),
                  "%2B"),
                Char(47),
                "%2F"),
              Char(32),
              "%20"),
            Char(45),
            "%2D"),
          Char(36),
          "%24"),
        Char(37),
        "%25"),
      Char(46),
      "%2E"
    ),
    "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"
  ),
  '<br /><h2 style="color: white; background-color: #AD2A1A; text-align: center;">This item was deleted from the catalog<br />within the past 13 months</h2>'
  ) AS INFO
FROM
  deleteditems
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE'
    ) ccodes
  ON deleteditems.ccode = ccodes.authorised_value
LEFT JOIN biblio
  ON deleteditems.biblionumber = biblio.biblionumber
LEFT JOIN deletedbiblio
  ON deleteditems.biblionumber = deletedbiblio.biblionumber
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED'
    ) damagedi
  ON damagedi.authorised_value = deleteditems.damaged
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST'
    ) losti
ON losti.authorised_value = deleteditems.itemlost
LEFT JOIN (
    SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN'
    ) deletedwithdrawni
ON deletedwithdrawni.authorised_value = deleteditems.withdrawn
WHERE
  deleteditems.barcode LIKE Concat("%", <<Enter barcode number>>, "%")
GROUP BY
  deleteditems.itemnumber

~~~

<mark>(Warning - if you copy and paste this report directly into your system - it will not work until you create the accessory reports and edit this SQL with the correct report numbers)</mark>

Yes.  I know.  It's a big report.  It's got a lot going on in it.

Here's what it does:

If you enter an item barcode number and that barcode number is in items.barcode or deleteditems.barcode, you will get a result that lists pertinent information about the item in a format that's easy to cut-and-paste into a document or an e-mail.

If the item has not been deleted, you'll get links to the item record, the bibliographic record, and 5 other reports.  A report for circulation history, a report for action log history, a report for the item's in-transit history, and reports for the title's request history and the item's request history.

If the item has been deleted, you'll get a link to the bibliographic record (unless the biblio has also been deleted) and a link to a report that will show you any accountlines where the item barcode number appears in the accountline description or note fields.

Here's why it does all of this:

I manage a Koha that is shared by 51 libraries and uses a courier service to move materials across 7400 square miles of north east Kansas.  Of these 51 libraries, the smallest serves a community of less than 200 and the largest serves a community of over 35,000.  The number of staff as well as their experience and education varies from library to library.  It is not uncommon for me to receive phone calls or e-mails saying, "I have an item with barcode number X.  I need to know more about it."

I can use Koha to find out a ton of things about an item.  I can tell you where it's been and when it was there.  I can tell you if it's still active in the system or if it's been deleted within the last 13 months (we run a script that purges the deleted items table of records that have been in deleteditems for more than 13 months).  I can tell you its circulation history, its requset history, and other things with just the barcode number.  But I have to go to a bunch of different places to find these things out.

I wrote this report to be a one-stop-shop for an item barcode number search.  If I get an e-mail saying "I have this barcode number and when I scan it I get __ happens.  Why does __ happen."

With this report, I can scan the barcode number once and if the barcode number is still active or has been deleted within the last 13 months, I get a result.  I don't have to run separate reports for deleted and non-deleted items.

If the item is still active, I can easily copy and paste item information into an e-mail, go to more specific item data, or run reports about circulation history, action log history, in transit history, or the item's request history.  And if it's been deleted within the last 13 months, I get a result that I can easily copy and past into an e-mail and a link to references in the accountlines notes and descriptions where the barcode number can be found.  And if it was deleted more than 13 months ago, I get no result.  In short, this report puts 90% of the things I need for answering questions about items into 1 spot.

----------

So, in order to help others understand how this report was built, I'm going to walk through all of the steps I took to build it, one step at a time.

## Step 1

I started with a report that looks at basic item information from the items and biblio table.

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  items.permanent_location,
  items.location,
  items.itype,
  items.ccode,
  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  items.notforloan,
  items.damaged,
  items.damaged_on,
  items.itemlost,
  items.itemlost_on,
  items.withdrawn,
  items.withdrawn_on
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber
WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

~~~

This report gets me the basic information I want for an item that is still in the system, but I really want something I can cut and paste into an e-mail that's easy to read and that will make as much sense to a brand new employee who's straight out of high school as to a library director with a Masters degreen and 35 years of experience.  

## Step 2

My next step is to add some sub-queries that will convert some of the codes into descriptions (i.e. instead of item type = "NVIDNEW" staff will item type = "Video (new)"; instead of itemlost = 2, staff will see itemlost = "Lost (more than 45 days overdue)."

If you don't know how to create sub-queries, please check out the video: <a href="https://youtu.be/iRBEvt4nDbU" target="_blank">SQL: Dates and Subqueries</a>

As a first sub-query example, this next sample code includes just a sub-query for getting the collection code description instead of just the collection code authorised value:

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  items.permanent_location,
  items.location,
  items.itype,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  items.notforloan,
  items.damaged,
  items.damaged_on,
  items.itemlost,
  items.itemlost_on,
  items.withdrawn,
  items.withdrawn_on
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'ccode') ccodes ON ccodes.authorised_value =
      items.ccode
WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

~~~

The two changes involve changing "items.ccode" into "ccodes.lib AS CCODE" and creating the following as a sub-query called "ccodes"

~~~SQL

SELECT
    authorised_values.category,
    authorised_values.authorised_value,
    authorised_values.lib
  FROM
    authorised_values
  WHERE
    authorised_values.category = 'ccode'

~~~

This sub-query gets a list of all of the collection codes in your system.  By left-joining it to the items table, I'm telling the report to grab the collection code description (if there is one) and use it instead of items.ccode which displays the collection code authorised value.  The description is usually a lot easier to read for inexperienced staff than the authorised value.  This is especially true of the numeric codes used for items.notforloan, items.itemlost, items.damaged, etc.

----------

That sample should get you a good idea of how to use a sub-query, so this next sample includes all of the fields where I want to use a sub-query instead of just a code:

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  notforloans.lib,
  damageds.lib AS DAMAGED,
  items.damaged_on,
  losts.lib AS LOST,
  items.itemlost_on,
  withdrawns.lib AS WITHDRAWN,
  items.withdrawn_on
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn
WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

~~~

An important thing to remember here is that I want all of these joins to be LEFT JOIN s.  This way if an item doesn't have a "Lost" status or a "Not for loan" status, etc., you'll still get a result.  If you do a simple JOIN, and there is no result in the sub-query, you won't get any result at all.

## Step 4

One of the things that would be nice is if the "Lost" and "Withdrawn" information and the lost_on and withdrawn_on dates appeared together instead of in separate columns.  That's why this next sample concatenates damaged and damaged_on; lost and lost_on; and withdrawn and withdrawn_on into three columns instead of six

If you don't know how to use the "Concat" function, please check out the video: <a href="https://youtu.be/xq9oQ1iP6c0" target="_blank">Concatenating and If Statements in Reports</a>

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  biblio.title,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  notforloans.lib,
  CONCAT(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  CONCAT(losts.lib, ' ', items.itemlost_on) AS LOST,
  CONCAT(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn
WHERE
  items.barcode LIKE CONCAT('%', <<Enter item barcode>>, '%')
GROUP BY
  items.itemnumber,
  biblio.biblionumber

~~~

This takes the report from 26 columns to 23 columns.  That's still a lot to cut and paste into an e-mail.

## Step

I know that if I want this report to be useful for library staff, I'm going to have to add some more title information.  The biblio.title field only includes data from the 245$a in the Marc record.  If I want 245$b, $n, $p, $h, and $c I can add those fields directly from the database if I've got my Koha to Marc mapping set up to do that.  Or I can just extract the data from the biblio_metadata table directly from the Marc records.  Since I started working with Koha before some of this information was able to be mapped, I tend to join to the biblio_metadata table and go that route.  This also uses CONCAT_WS which is mentioned in the video at <a href="https://youtu.be/xq9oQ1iP6c0" target="_blank">Concatenating and If Statements in Reports.</a>

To see a video about extracting data from Marc records in the biblio_metadata table, please check out the video: <a href="https://youtu.be/VrjGxaoRCIw" target="_blank">SQL: ExtractValue.</a>

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  items.onloan,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn INNER JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode>>, '%')
GROUP BY
  notforloans.lib,
  damageds.lib,
  losts.lib,
  withdrawns.lib,
  items.itemnumber,
  biblio.biblionumber

~~~

## Step

At this point I also know that, if the item is checked out, I'm going to want to have a quick way to go to the borrower's record.  Unfortunately there isn't anything in the item record that tells me who an item is checke out to.  In order to get information about who an item is checked out to now, I need to link to the issues table, but I only want a result if the item is actually checked out.  This leads us to our first if/then statement of the report.  It also leads to the first HTML link built into the report.

If you don't know how to create an if/then statement, please check out the video: <a href="https://youtu.be/xq9oQ1iP6c0" target="_blank">Concatenating and If Statements in Reports.</a>

If you don't know how to create a link in a report, please check out the video: <a href="https://youtu.be/71ETEh_cFH0" target="_blank">Links in Reports</a>

I'm also going to switch from using items.onloan to issues.due date at this point.  No special reason for that - just because.

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,
  If(
    issuesi.date_due IS NULL,
    "-",
    Concat(
      "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
      issuesi.borrowernumber,
      "' target='_blank'>go to the borrower's account</a>"
    )
  ) AS LINK_TO_BORROWER
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn INNER JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode>>, '%')
GROUP BY
  issuesi.date_due,
  notforloans.lib,
  damageds.lib,
  losts.lib,
  withdrawns.lib,
  items.itemnumber,
  biblio.biblionumber

~~~

This piece:

~~~SQL

Concat(
  "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
  issuesi.borrowernumber,
  "' target='_blank'>go to the borrower's account</a>"
)

~~~

Creates the link to the borrower's record by combining some HTML with the borrower's ID number while the if/then statement sourrounding it tells the report to that, if there is no borrower number to report a hyphen instead of displaying an HTML link that won't work properly.

## Step

At this point I'm going to add in data from the branch transfers table so I can tell if the item is in transit between two libraries.

This involves another sub-query linking out to the branchtransfers table and listing items where branchtransfers.datearrived is null.  Once we have that we can get information about the from and to branches and concatenate them into the results.  Again, this is done through a left-join so that, if the item is not in transit, there will still be a result.  And I'm building it with an if/then statement so that, if the items isn't in transit, there will just be a hyphen instead of empty branchcodes saying "to" and "from."

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,
  If(
    transfersi.frombranch IS NULL,
    "-",
    Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent)
  ) AS TRANSFER,
  If(
    issuesi.date_due IS NULL, "-",
    Concat(
      "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
      issuesi.borrowernumber,
      "' target='_blank'>go to the borrower's account</a>"
    )
  ) AS LINK_TO_BORROWER
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn INNER JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber LEFT JOIN
  (SELECT
      branchtransfers.itemnumber,
      branchtransfers.frombranch,
      branchtransfers.datesent,
      branchtransfers.tobranch,
      branchtransfers.datearrived
    FROM
      branchtransfers
    WHERE
      branchtransfers.datearrived IS NULL) transfersi ON transfersi.itemnumber =
      items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode 0003008200544>>, '%')
GROUP BY
  issuesi.date_due,
  notforloans.lib,
  damageds.lib,
  losts.lib,
  withdrawns.lib,
  items.itemnumber,
  biblio.biblionumber

~~~

## Step

I also know that at some point I'm going to want to add labels to the data, so I'm going to do a single sample with the link that was just created:

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title,
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata, '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,
  Concat("Link to borrower: ",
    If(
      issuesi.date_due IS NULL,
      "-",
      Concat(
        "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
        issuesi.borrowernumber,
        "' target='_blank'>go to the borrower's account</a>"
      )
    )
  ) AS LINK_TO_BORROWER
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn INNER JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode>>, '%')
GROUP BY
  issuesi.date_due,
  notforloans.lib,
  damageds.lib,
  losts.lib,
  withdrawns.lib,
  items.itemnumber,
  biblio.biblionumber

~~~

By wrapping "Concat("Link to borrower: ", )" around the existing link to the borrower's information, I'm creating a lable that will be useful when we get to the future steps.

~~~SQL

Concat("Link to borrower: ",
  If(
    issuesi.date_due IS NULL,
    "-",
    Concat(
      "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",
      issuesi.borrowernumber,
      "' target='_blank'>go to the borrower's account</a>"
    )
  )
) AS LINK_TO_BORROWER

~~~

## Step

Next I'm going to add in links with labels to other reports.  The important thing here is to have the reports on your system and to update the report numbers from what I have in my system to what you have in your system.  The reports I'm using are:

Item circ history: 2785 - <a href="" target="_blank">Click here for report 2785</a>
Item action log history: 3342 - <a href="" target="_blank">Click here for report 3342</a>
Item in transit history: 2784 - <a href="" target="_blank">Click here for report 2784</a>
Request history on this title: 3039 - <a href="" target="_blank">Click here for report 3039</a>
Request history on this item: 3039 - <a href="" target="_blank">Click here for report 3039</a>

I realize here that koha-US doesn't have a video on how to link results from one report to another report.  It's actually just like creating any other link as described in the video on links or at <a href="https://wiki.koha-community.org/wiki/SQL_Reports_Library#Links" target="_blank">the Koha Wiki's SQL library.</a>  The difference is that the URL will start with "/cgi-bin/koha/reports/guided_reports.pl?reports=", followed by the report number, followed by the runtime parameters you might need for the report.

For example, to run my "Item action log history" report you need to concatenate:

>"<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params="

to start the URL along with the itemnumber from the database followed by:

>"' target='_blank'>see action log history</a>"

to finish the HTML.

In this case the item number fills in the run time parameter the report is asking for.

Generally speaking, the best way to figure out how to concatenate report data into a report URL is to run that report with a sample set of runtime parameters and then copy and paste the URL into a text editor in order to figure out where the varialbes will go.  For example, if I run my "Request history on this item" with item barcode number 0003008200544, the resulting url is:

~~~HTML

/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&param_name=Choose+pickup+library%7CLBRANCH&sql_params=%25&param_name=Choose+request+status%7CLHOLDACT&sql_params=%25&param_name=Choose+request+progress%7CLHOLDPROG&sql_params=%25&param_name=Choose+suspended+status%7CLHOLDSUS&sql_params=%25&param_name=Enter+library+card+number+or+a+%25+symbol&sql_params=%25&param_name=Enter+title+biblio+number+or+a+%25+symbol&sql_params=%25&param_name=Enter+item+barcode+number+or+a+%25+symbol&sql_params=0003008200544

~~~

and I can see the barcode number as the last 13 digits of that URL so I know that I would want to use everything up to the barcode number and then concatenate in items.barcode where the barcode number falls in this URL.

Anyway, there are plenty of examples to look at in this next version of the report:

~~~SQL

SELECT
  items.homebranch,
  items.holdingbranch,
  permanent_locss.lib AS PERM_LOCATION,
  locss.lib AS LOCATION,
  itemtypess.description AS ITYPE,
  ccodes.lib AS CCODE,
  items.itemcallnumber,
  biblio.author,
  Concat_Ws(' ',
    biblio.title, ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="h"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="b"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="n"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="p"]'),
    ExtractValue(biblio_metadata.metadata,  '//datafield[@tag="245"]/subfield[@code="c"]')
  ) AS FULL_TITLE,
  items.barcode,
  items.itemnotes,
  items.itemnotes_nonpublic,
  items.issues,
  items.renewals,
  items.dateaccessioned,
  items.datelastborrowed,
  items.datelastseen,
  items.timestamp,
  issuesi.date_due,
  notforloans.lib,
  Concat(damageds.lib, ' ', items.damaged_on) AS DAMAGED,
  Concat(losts.lib, ' ', items.itemlost_on) AS LOST,
  Concat(withdrawns.lib, ' ', items.withdrawn_on) AS WITHDRAWN,
  Concat(
    "Link to borrower: ",
    If(
      issuesi.date_due IS NULL,
      "-",
      Concat(
        "<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=",  
        issuesi.borrowernumber,
        "' target='_blank'>go to the borrower's account</a>"
      )
    )
  ) AS LINK_TO_BORROWER,
  Concat(
    "Link to title: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the bibliographic record</a>"
    )
  ) LINK_TO_TITLE,
  Concat(
    "Link to item: ",
    Concat(
      "<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=",
      items.itemnumber,
      "&biblionumber=",
      biblio.biblionumber,
      "' target='_blank'>go to the item record</a>"
    )
  ) AS LINK_TO_ITEM,
  Concat(
    "Item circ history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=",
      items.barcode,
      "' target='_blank'>see item circ history</a>"
    )
  ) AS CIRC_HISTORY_REPORT,
  Concat(
    "Item action log history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=",
      items.itemnumber,
      "' target='_blank'>see action log history</a>"
    )
  ) AS ACTION_LOG,     
  Concat(
    "Item in transit history: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=",
      items.barcode,
      "' target='_blank'>see item transit history</a>"
    )
  ) AS_IN_TRANSIT,
  Concat(
    "Request history on this title: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      biblio.biblionumber,
      "&sql_params=%25' target='_blank'>see title's request history</a>"
    )
  ) AS REQUEST_HISTORY_TITLE,
  Concat(
    "Request history on this item: ",
    Concat(
      "<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=",
      items.barcode,
      "' target='_blank'>see item's request history</a>"
    )
  ) AS REQUEST_HISTORY_ITEM
FROM
  items JOIN
  biblio ON items.biblionumber = biblio.biblionumber LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') permanent_locss ON
      permanent_locss.authorised_value = items.permanent_location LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOC') locss ON locss.authorised_value =
      items.location LEFT JOIN
  (SELECT
      itemtypes.itemtype,
      itemtypes.description
    FROM
      itemtypes) itemtypess ON itemtypess.itemtype = items.itype LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'CCODE') ccodes ON ccodes.authorised_value =
      items.ccode LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'NOT_LOAN') notforloans ON
      notforloans.authorised_value = items.notforloan LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'DAMAGED') damageds ON
      damageds.authorised_value = items.damaged LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'LOST') losts ON losts.authorised_value =
      items.itemlost LEFT JOIN
  (SELECT
      authorised_values.category,
      authorised_values.authorised_value,
      authorised_values.lib
    FROM
      authorised_values
    WHERE
      authorised_values.category = 'WITHDRAWN') withdrawns ON
      withdrawns.authorised_value = items.withdrawn INNER JOIN
  biblio_metadata ON biblio_metadata.biblionumber = biblio.biblionumber
  LEFT JOIN
  (SELECT
      issues.itemnumber,
      issues.date_due,
      issues.borrowernumber
    FROM
      issues) issuesi ON issuesi.itemnumber = items.itemnumber
WHERE
  items.barcode LIKE Concat('%', <<Enter item barcode 0003008200544>>, '%')
GROUP BY
  issuesi.date_due,
  notforloans.lib,
  damageds.lib,
  losts.lib,
  withdrawns.lib,
  items.itemnumber,
  biblio.biblionumber

~~~

## Step



























, please check out the video: <a href="" target="_blank"></a>
