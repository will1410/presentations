#Item Dashboard report

##How to create the item dashboard report

The SQL for this report is kind of crazy:

<details>
  <summary>SQL for Item dashboard</summary>

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
          Concat("Lost status: ", If(items.itemlost = 0, "-", If(items.itemlost IS NULL, "-", Concat(losti.lib, " on ", items.itemlost_on)))),
          Concat("Withdrawn status: ", If(items.withdrawn = 0, "-", If(items.withdrawn IS NULL, "- ", Concat(withdrawni.lib, " on ", items.withdrawn_on)))),
          Concat("<br /> In transit from ", If(transfersi.frombranch IS NULL, "-", Concat(transfersi.frombranch, " to ", transfersi.tobranch, " since ", transfersi.datesent))),
          Concat("<br />Link to borrower: ", If(issuesi.date_due IS NULL, "-", Concat("<a href='/cgi-bin/koha/circ/circulation.pl?borrowernumber=", issuesi.borrowernumber, "' target='_blank'>go to the borrower's account</a>"))),
          Concat("Link to title: ", Concat("<a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=", biblio.biblionumber, "' target='_blank'>go to the bibliographic record</a>")),
          Concat("Link to item: ", Concat("<a href='/cgi-bin/koha/catalogue/moredetail.pl?itemnumber=", items.itemnumber, "&biblionumber=", biblio.biblionumber, "' target='_blank'>go to the item record</a>")),
          Concat("Item circ history: ", Concat("<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2785&phase=Run+this+report&param_name=Enter+item+barcode+number&sql_params=", items.barcode, "' target='_blank'>see item circ history</a>")),
          Concat("Item action log history: ", Concat("<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3342&phase=Run+this+report&param_name=Enter+item+number&sql_params=", items.itemnumber, "' target='_blank'>see action log history</a>")),     
          Concat("Item in transit history: ", Concat("<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=2784&phase=Run+this+report&sql_params=", items.barcode, "' target='_blank'>see item transit history</a>")),
          Concat("Request history on this title: ", Concat("<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=", biblio.biblionumber, "&sql_params=%25' target='_blank'>see title's request history</a>")),
          Concat("Request history on this item: ", Concat("<a href='/cgi-bin/koha/reports/guided_reports.pl?reports=3039&phase=Run+this+report&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=%25&sql_params=", items.barcode, "' target='_blank'>see item's request history</a>")),
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
        Concat('At the time of its deletion on:  <ins><strong>', deleteditems.timestamp, "<br /></strong></ins> this item's information was as follows:<br />"),
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
        Concat("<br />Damaged status: ", If(deleteditems.damaged = 0, "-", If(deleteditems.damaged IS NULL, "-", damagedi.lib))),
        Concat("Lost status: ", If(deleteditems.itemlost = 0, "-", If(deleteditems.itemlost IS NULL, "-", Concat(losti.lib, " on ", deleteditems.itemlost_on)))),
        Concat("Withdrawn status: ", If(deleteditems.withdrawn = 0, "-", If(deleteditems.withdrawn IS NULL, "- ", Concat(deletedwithdrawni.lib, " on ", deleteditems.withdrawn_on)))),
        If(biblio.biblionumber IS NULL, "<br />-- Bibliographic record has been deleted --", Concat("<br /><a href='/cgi-bin/koha/catalogue/detail.pl?biblionumber=", biblio.biblionumber, "' target='_blank'>Go to the bibliographic record</a>")),
        Concat("<br /><a href='/cgi-bin/koha/reports/guided_reports.pl?phase=Run+this+report&reports=3009&sql_params=", Replace(Replace(Replace(Replace(Replace(Replace(Replace(deleteditems.barcode, Char(43), "%2B"), Char(47), "%2F"), Char(32), "%20"), Char(45), "%2D"), Char(36), "%24"), Char(37), "%25"), Char(46), "%2E"), "&limit=50' target='_blank'>Search payment and fee notes and descriptions for this item barcode number</a>"),
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

</details>

Yes.  I know.  It's a big report.  It's got a lot going on in it.

Here's what it does:

If you enter an item barcode number and that barcode number is in items.barcode or deleteditems.barcode, you will get a result that list pertinent information about the item in a format that's easy to cut-and-paste into a document or an e-mail.

If the item has not been deleted, you'll get links to the item record, the bibliographic record, and 5 other reports.  A report for circulation history, a report for action log history, a report for the item's in-transit history, and reports for the title's request history and the item's request history.

If the item has been deleted, you'll get a link to the bibliographic record (unless the biblio has also been deleted) and a link to a report that will show you any accountlines where the item barcode number appears in the accountline description or note fields.

Here's why it does all of this:

I manage a Koha that is shared by 51 libraries and uses a courier service to move materials across
