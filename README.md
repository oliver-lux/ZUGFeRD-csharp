Part of the ZUGFeRD community:
https://github.com/zugferd

# Introduction
The ZUGFeRD library allows to create XML files as required by German electronic invoice initiative ZUGFeRD as well invoices in the successor Factur-X. One special profile of Factur-X is the German XRechnung format.
The library is meant to be as simple as possible, however it is not straight forward to use as the resulting XML file contains a complete invoice in XML format. Please take a look at the ZUGFeRD-Test project to find sample creation code. This code creates the same XML file as shipped with the ZUGFeRD information package.

A description of the library can be found here:

http://www.s2-industries.com/wordpress/2013/11/creating-zugferd-descriptors-with-c/

# License
Subject to the Apache license http://www.apache.org/licenses/LICENSE-2.0.html

# Installation
Just use nuget or Visual Studio Package Manager and download 'ZUGFeRD-chsarp'.

You can find more information about the nuget package here:

https://www.nuget.org/packages/ZUGFeRD-csharp/

# Building on your own
Prerequisites:
* Visual Studio >= 2017
* .net Framework >= 4.6.1 (for .net Standard 2.0 support)

Open ZUGFeRD/ZUGFeRD.sln solution file. Chose Release or Debug mode and hit 'Build'. That's it.

For running the demo, open ZUGFeRD-Test/ZUGFeRD-Test.sln and run the application.

# Creating invoices
Central class for users is class InvoiceDescriptor.
This class does not only allow to read and set all ZUGFeRD attributes and structures but also allows to load and save ZUGFeRD files.

Using InvoiceDescriptor to create invoices is straight forward:

```csharp
InvoiceDescriptor desc = InvoiceDescriptor.CreateInvoice("471102", new DateTime(2013, 6, 5), CurrencyCodes.EUR, "GE2020211-471102");
desc.Profile = Profile.Comfort;
desc.ReferenceOrderNo = "AB-312";
desc.AddNote("Rechnung gemäß Bestellung Nr. 2013-471331 vom 01.03.2013.");
desc.AddNote("Es bestehen Rabatt- und Bonusvereinbarungen.", "AAK");
desc.SetBuyer("Kunden Mitte AG", "69876", "Frankfurt", "Kundenstraße", "15", "DE", "88", "4000001987658");
desc.AddBuyerTaxRegistration("DE234567890", "VA");
desc.SetBuyerContact("Hans Muster");
desc.SetSeller("Lieferant GmbH", "80333", "München", "Lieferantenstraße", "20", "DE", "88", "4000001123452");
desc.AddSellerTaxRegistration("201/113/40209", "FC");
desc.AddSellerTaxRegistration("DE123456789", "VA");
desc.SetBuyerOrderReferenceDocument("2013-471331", new DateTime(2013, 03, 01));
desc.SetDeliveryNoteReferenceDocument("2013-51111", new DateTime(2013, 6, 3));
desc.ActualDeliveryDate = new DateTime(2013, 6, 3);
desc.SetTotals(202.76m, 5.80m, 14.73m, 193.83m, 21.31m, 215.14m, 50.0m, 165.14m);
desc.AddApplicableTradeTax(9.06m, 129.37m, 7m, "VAT", "S");
desc.AddApplicableTradeTax(12.25m, 64.46m, 19m, "VAT", "S");
desc.SetLogisticsServiceCharge(5.80m, "Versandkosten", "VAT", "S", 7m);
desc.SetTradePaymentTerms("Zahlbar innerhalb 30 Tagen netto bis 04.07.2013, 3% Skonto innerhalb 10 Tagen bis 15.06.2013", new DateTime(2013, 07, 04));

desc.Save("output.xml");
```

# Using ZUGFeRD 1.x, ZUGFeRD 2.x and XRechnung
In order to load ZUGFeRD files, you call InvoiceDescriptor.Load(), passing a file path like this:

```csharp
InvoiceDescriptor descriptor = InvoiceDescriptor.Load("zugferd.xml");
```

alternatively, you can pass an open stream object:

```csharp
Stream stream = new FileStream("zugferd.xml", FileMode.Open, FileAccess.Read);
InvoiceDescriptor descriptor = InvoiceDescriptor.Load(stream);
```

The library will automatically detect the ZUGFeRD version of the file and parse accordingly. As of today (2020-07-05), parsing ZUGFeRD 2.x is not yet finished.
The lifecycle of the stream is not influenced by the ZUGFeRD library, i.e. the library expects an open stream and will not close if after reading from it.

For saving ZUGFeRD files, use InvoiceDescriptor.Save(). Here, you can also pass a stream object:

```csharp
InvoiceDescriptor descriptor = InvoiceDescriptor.CreateInvoice(......);

// fill attributes and structures

FileStream stream = new FileStream(filename, FileMode.Create, FileAccess.Write);
descriptor.Save(stream, ZUGFeRDVersion.Version1, Profile.Basic);
stream.Flush();
stream.Close();            
```

As you see, the libary does not influence the lifecycle of the stream, i.e. it is not automatically closed by the library. Just as opening the stream, flushing and closing is the duty of the calling function.

Alternatively, you can pass a file path:

```csharp
InvoiceDescriptor descriptor = InvoiceDescriptor.CreateInvoice(......);

// fill attributes and structures

descriptor.Save("zugferd.xml", ZUGFeRDVersion.Version1, Profile.Basic);          
```

Optionally, you can pass the ZUGFeRD version to use, default currently is version 1.x, e.g.:

```csharp
InvoiceDescriptor descriptor = InvoiceDescriptor.CreateInvoice(......);

// fill attributes and structures


descriptor.Save("zugferd-v1.xml", ZUGFeRDVersion.Version1, Profile.Basic); // save as version 1.x
descriptor.Save("zugferd-v2.xml", ZUGFeRDVersion.Version2, Profile.Basic); // save as version 2.0
descriptor.Save("zugferd-v2.xml", ZUGFeRDVersion.Version21, Profile.Basic); // save as version 2.1
```

# Special features for XRechnung
In general, creating XRechnung files is straight forward and just like creating any other ZUGFeRD version and profile:

```csharp
descriptor.Save("xrechnung.xml", ZUGFeRDVersion.Version21, Profile.XRechnung); // save as XRechnung
```

However, XRechnung comes with some special features. One of these features is the ability to embed binary files as attachments to the xrechnung.xml document:

```csharp
InvoiceDescriptor desc = _createInvoice();
byte[] data = System.IO.File.ReadAllBytes("my-calculation.xlsx");
desc.AddAdditionalReferencedDocument(
    issuerAssignedID: "calculation-sheet",
    typeCode: AdditionalReferencedDocumentTypeCode.ReferenceDocument,
    name: "Calculation as the basis for the invoice",
    attachmentBinaryObject: data,
    filename: "my-calculation.xlsx");

desc.Save("xrechnung.xml", ZUGFeRDVersion.Version21, Profile.XRechnung);            
```

The resulting xrechnung.xml file will then contain the calculation file content, encoded in base64.
Please note that there are only few mime types supported by the XRechnung standard. At the time of writing this tutorial, those are listed here:
https://projekte.kosit.org/xrechnung/xrechnung/-/issues/59

- application/pdf
- image/png
- image/jpeg
- text/csv
- application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
- application/vnd.oasis.opendocument.spreadsheet
- application/xml

# Support for profiles
The library contains support for all profiles that are supported by the ZUGFeRD formats:

| Profile         	| Version1 	| Version2 	| Version21 	|
|-----------------	|----------	|----------	|-----------	|
| MINIMUM         	|          	| X        	| X         	|
| BASIC WL        	|          	| X        	| X         	|
| BASIC           	| X        	| X        	| X         	|
| COMFORT/EN16391 	| X        	| X        	| X         	|
| XRECHNUNG       	|          	|          	| X         	|
| EXTENDED        	| X        	| X        	| X         	|

please note that version 1 implementation of the library is not strict, i.e. it will output all information available into the invoice xml, regardless of the profiles that is used. Reading various files with different profiles will generate the correct output.

Beginning with version 2.1, the output is corresponding exactly to the profile that is used.

If you want to write the invoice xml with a certain ZUGFeRD version and a certain profile, make sure to use the parameters of the Save method:

```csharp
descriptor.Save("zugferd-v1.xml", ZUGFeRDVersion.Version1, Profile.Basic); // save as version 1.x, profile Basic
descriptor.Save("zugferd-v2.xml", ZUGFeRDVersion.Version2, Profile.Basic); // save as version 2.0, profile Basic
descriptor.Save("zugferd-v2.xml", ZUGFeRDVersion.Version21, Profile.Basic); // save as version 2.1, profile Basic
descriptor.Save("zugferd-v2.xml", ZUGFeRDVersion.Version21, Profile.XRechnung); // save as version 2.1, profile XRechnung
```

# Extracting xml attachments from pdf files
I'm getting asked frequently on how one can extract the ZUGFeRD/ Factur-X/ XRechnung attachment from existing PDF files.

There is a nice article on stackoverflow on how this can be achieved using itextsharp:

https://stackoverflow.com/a/6334252

and this one covers the same with itext7 which is the successor of itextsharp:

https://stackoverflow.com/a/37804285

# Thanks
* The solution is used in CKS.DMS and supported by CKSolution: 
  http://www.cksolution.de
* ZUGFeRD 2.1 implementation was done by www.netco-solution.de and used in netCo.Butler

# Links
You can find more information about ZUGFeRD here:
http://www.ferd-net.de/

