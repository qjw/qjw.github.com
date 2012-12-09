---
layout: post
title:  perl使用Validator::Schema校验xml
category: bash
---        

**aptitude install libxml-validator-schema-perl**

    use XML::SAX::ParserFactory;
    use XML::Validator::Schema;

    #
    # create a new validator object, using foo.xsd
    #
    $validator = XML::Validator::Schema->new(file => 'foo.xsd');

    #
    # create a SAX parser and assign the validator as a Handler
    #
    $parser = XML::SAX::ParserFactory->parser(Handler => $validator);

    #
    # validate foo.xml against foo.xsd
    #
    eval { $parser->parse_uri('foo.xml') };
    die "File failed validation: $@" if $@;

##参考
1. <http://search.cpan.org/~samtregar/XML-Validator-Schema-1.08/Schema.pm>