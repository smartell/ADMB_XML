ADMB_XML
========

xml interface for I/O of ADMB types

Purpose
-------

To create an xml interface for ADMB that permits storing all data, model structure, control flags, estimation phases, and parameter bounds in a single file. The intent is for the xml tree to contain complete documentation for every parameter estimation run using an ADMB application.


Advantages of using xml: 

1. Order of named variable nodes in file is unimportant as long as the integrity of the tree structure is respected.

2. Easily viewed in browser using a xsl style sheet.

3. Speed. The xml tree is only read from the file once. Data elements can be extracted (or updated) in any order as needed without reading the file again.

4. Xml trees can be extended to accomodate whatever is needed (the 'x' in xml).

5. Comments are easily added to xml files; e.g.  `<!-- this is a s comment -->`.

6. The format of xml files is very relaxed; spaces and extra lines can be inserted.

7. R and other computing packages support xml.

8. Could be used as a basis for maintaining a data base of alternate fits for model selection.



Disadvantages of using xml:

1. Xml files can be tedious to edit, but is unusual to need to edit a large object such as an array.
 
2. Creating an initial xml file can be problematical (but see example tpl).


Requirements
------------

ADMB_XML uses the [libxml2](http://www.xmlsoft.org/ "libxml2") library which is available for many computing platforms including Linux, Windows, CygWin, MacOS, MacOS X, and others.

It has been tested on Ubuntu 12.04 64bit using g++ 4.8.0.



pella-xml.tpl example
-------------

This example tpl is Arni Magnusson's [Pella](http://www.admb-project.org/examples/fisheries/pella/ "Pella") example with the alb data. One FUNCTION has been added to write data and parameters into an xml tree.

    FUNCTION saveXMLFile
      ADMB_XMLDoc xml;
      int ret = 0;

      if (fabs(neglogL) <= 0.0)
        xml.allocate("FIT","pella-xml","Starting Values", "pella-xml.x00");
      else
      {
        xml.allocate("FIT","pella-xml","Final Estimates", "pella-xml.x01");
        ret = xml.createXMLcomment("fit statistics");
        ret = xml.createXMLelement(neglogL);
      } 

      ret = xml.createXMLelement(nc,"Catch Data Years");
      ret = xml.createXMLelement(Cdata,"Catch Data");
      ret = xml.createXMLelement(ni,"Survey Data Years");
      ret = xml.createXMLelement(Idata,"Survey Data");

      ret = xml.createXMLelement(logr,"log growth rate");
      ret = xml.createXMLelement(logk,"log carrying capacity");
      ret = xml.createXMLelement(loga,"log a");
      ret = xml.createXMLelement(logp,"log p");
      ret = xml.createXMLelement(logq,"log q");
      ret = xml.createXMLelement(logsigma,"log sigma");

      ret = xml.write();

This function is called twice: once at the end of PRELIMINARY_CALCS section to record the data and starting values of the parameters, and once in the REPORT_SECTION to record the statistics about the fit, parameter estimates and the data. The `ADMB_XMLDoc::createXMLelement(...)` member function is overloaded for many of the data types generated by `tpl2cpp` (there are few that I missed).


The two output files `pella-xml.x00` and `pella-xml.x01` can be viewed either with a text editor or with a bowser (Firefox works fine) using the supplied xml stylesheet `ADMB.xsl`.

xpella.tpl example
-------------

This example tpl is another adaptation Arni Magnusson's [Pella](http://www.admb-project.org/examples/fisheries/pella/ "Pella") example with the alb data. All `init_` variables are declared using the xml document tree read in from the file xpella.xml. Portions of the delcarations from the DATA_SECTION and PARAMETER_SECTION are shown below:


    DATA_SECTION
      // Read data xml file
      init_xml_doc xml
      init_int nc(xml)
      init_matrix Cdata(xml)
      init_int ni(xml)
      init_matrix Idata(xml)
      .
      .
      .
    
    PARAMETER_SECTION
      // Estimated
      init_bounded_number logr(xml)
      init_bounded_number logk(xml)
      init_bounded_number loga(xml)
      init_bounded_number logp(xml)
      init_bounded_number logq(xml)
      init_bounded_number logsigma(xml)
      .
      .
      .

The saveXMLFile from pella-xml.tpl is used to write data and parameters into an xml tree.


Next Steps
----------

The recent insights from Dave Fournier, Arni Magunsson and Steve Martell have been invaluable for modifying the lex code and writing allocate(ADMB_XMLDoc& node) functions. Interested users should test this stuff and proved feedback on what is useful, what is not so useful, and what should be added.

1. Add lex and C++ code to specify file name for xml input in DATA_SECTION
2. Allocate functions need to be written for more ADMB types.
3. The format of the xml file generated by the ADMB_XMLDoc class is valid xml, but could be changed to improved readibility.


So What's in the xml files anyway
-------

- The entire xml tree is contained in a single document indicated by the tag "FIT" with an `id` field set at the time of tree creation.
- Within the FIT tag, every variable is in a xml node idendified by a tag idenfied by the name of the variable used in the tpl, a category indicating whether it is a constant type ("CON"), a variable type ("VAR"), or a likelihood value ("STAT").
- In addition to the above every variables contains a "value" tag containing the numerical value of the variable at time of data input and a "title" tag which contains a "user-friendly" definition of the variable.
- Variables may be scalar quantities, vectors or matrices. Vector and matrix objects contain and "index" tag which in turn contains tags for the upper and lower bounds of the object index or indices. Matrices are representd in row order from nrl to nrh.
- Variable objects contain additional tags. The "bounds" tag contains the minimum and maximum value  of the variable. The "phase" tag contains the estimation phase for the variable. In addition there is an "active" tag which indicates whether the parameter was active during estimation.
