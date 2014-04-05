---
layout: post
title:  perl使用XML::Libxml读写xml
category: bash
---        

**aptitude install libxml-libxml-perl**

        
    #!/usr/bin/perl

    # 注意@需要转义
    # count需要使用findnode
    
    use XML::LibXML;

    my $parser = XML::LibXML->new();
    my $doc = $parser->parse_file("a.xml");

    # 枚举节点
    my @result = $doc->findnodes("/dir/d");
    foreach $nod (@result) 
    {
            print $nod->to_literal . "\n";
    }

    print "\n";

    # 读取属性
    my @result = $doc->findnodes("/dir/d/\@n");
    foreach $nod (@result) 
    {
            print $nod->to_literal . "\n";
    }

    print "\n";

    # 过滤
    my @result = $doc->findnodes("/dir/d[\@n='b']");
    foreach $nod (@result) 
    {
            print $nod->to_literal . "\n";
    }
    
    print "\n";

    # count
    my $result = $doc->findvalue("count(/dir/d)");
    print $result . "\n";
    
----

    #!/usr/bin/perl

    use XML::LibXML;

    my $parser = XML::LibXML->new();
    my $doc = $parser->parse_file("a.xml");

    # 删除节点
    my @result = $doc->findnodes("/dir/d");
    foreach $nod (@result) 
    {
            my $parent = $nod->parentNode;
            $parent->removeChild($nod);
    }

    print "\n";

    # 过滤删除
    my @result = $doc->findnodes("/dir/f[\@n='cc']");
    foreach $nod (@result) 
    {
            my $parent = $nod->parentNode;
            $parent->removeChild($nod);
    }

    print "\n";

    # 删除属性
    my @result = $doc->findnodes("/dir/f");
    foreach $nod (@result) 
    {
            $nod->removeAttribute("n");
    }
    print $doc->serialize(serialize_c14n);

    # 删除节点
    my @result = $doc->findnodes("/dir/d/text()");
    foreach $nod (@result) 
    {
            $nod->setData("new value");
    }
    
##参考
1. <http://search.cpan.org/~shlomif/XML-LibXML-2.0014/lib/XML/LibXML/XPathContext.pod>
1. <http://search.cpan.org/~shlomif/XML-LibXML-2.0014/lib/XML/LibXML/Node.pod>
1. <http://search.cpan.org/dist/XML-LibXML/lib/XML/LibXML/Element.pod>
1. <http://search.cpan.org/~shlomif/XML-LibXML-2.0014/lib/XML/LibXML/Text.pod>
1. <http://stackoverflow.com/questions/8411684/xmllibxml-replace-element-value>