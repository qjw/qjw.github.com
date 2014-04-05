---
layout: post
title:  perl使用XML::Libxslt转换xml
category: bash
---        

**aptitude install libxml-libxslt-perl**

    use XML::LibXSLT;
    use XML::LibXML;

    my $xslt = XML::LibXSLT->new();

    my $source = XML::LibXML->load_xml(location => 'foo.xml');
    my $style_doc = XML::LibXML->load_xml(location=>'bar.xsl', no_cdata=>1);

    my $stylesheet = $xslt->parse_stylesheet($style_doc);

    my $results = $stylesheet->transform($source);

    print $stylesheet->output_as_bytes($results);

##参考
1. <http://search.cpan.org/~shlomif/XML-LibXSLT-1.79/LibXSLT.pm>