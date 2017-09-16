# Data files of German Decompounder for Apache Lucene / Apache Solr / Elasticsearch #

This project was started to offer German decompounding out of box for users
of Apache Lucene, Apache Solr, or Elasticsearch. The problem with the data files is
their license, so be careful when packaging them. Apache Lucene is an Apache v2.0
licensed project, so the data files cannot be shipped together with the distribution.

For decompounding German words, the recommended approach is the following:

* First use a hyphenator to create syllables of the input tokens. Of course this does
  way too much. If we would index syllables the user would match a lot of wrong stuff.
  The hyphenator rules are used in many word processor programs (e.g., Open Office or
  Latex). They are provided here in the format of an XML file for Apache FOPs (Formatting
  Objects Processor, taken from https://sourceforge.net/projects/offo/). Those files
  can be read by Lucene's `HyphenationCompoundWordTokenFilter` to do the hyphenation.
  Be sure to use the files of offo-hyphenation v1.2, not newer (2.x) ones (Lucene can't
  read them)!
* The second step is therefore to take the syllables and form words out of them again.
  The Lucene `HyphenationCompoundWordTokenFilter` can do this based on a dictionary.
  This project here mainly provides the dictionary to do this (see below). As the
  dictionary solely provides parts of compound words (not the compounds itsself),
  it is important to use the `onlyLongestMatch` setting of the token filter,
  otherwise you might get wrong decompounding results (especially as the dictionary
  also contains very short words).
* The third step is stemming the full word (the token filter keep the original by
  default) and also its parts. You should use the _light_ German stemmer (not the
  minimal), because the decompounded parts have lots of filler characters that should
  be removed by the stemmer. The minimal stemmer is not able to do this.
  As decompounding is no longer a minimal approach, you may consider to use a separate
  lucene field only using the minimal stemmer but not doing decompounding for
  preferring exact matches in your search.

The dictionary file [dictionary-de.txt](dictionary-de.txt) is developed here and was
created based on the fabulous data by Björn Jacke: https://www.j3e.de/ispell/igerman98/

I used his large and high quality dictionary to make a dictionary file only containing
the parts of German compounds. The dictionary therefore is not large, it contains
about 14,500 tokens, that are commonly used to form compounds. The dictionary does
*not* contain the compounds, only the parts that are used to create them.
The dictionary was lowercased and the umlauts restored to their UTF-8 representation.

*Keep in mind:* The files provided here are for *new* German orthography (since 1998)!

## Apache Solr example ##

Here is a config example for Apache Solr. To use it put the two data files
into the core's config directory's `lang` subfolder. After that you can add the
following definition to your Solr schema:

```xml
<!-- German -->
<fieldType name="text_de" class="solr.TextField" positionIncrementGap="100">
  <analyzer> 
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.HyphenationCompoundWordTokenFilterFactory" hyphenator="lang/de_DR.xml"
      dictionary="lang/dictionary-de.txt" onlyLongestMatch="true" minSubwordSize="4"/>
    <filter class="solr.GermanNormalizationFilterFactory"/>
    <filter class="solr.GermanLightStemFilterFactory"/>
  </analyzer>
</fieldType>
```

Important: Use the analyzer for both indexing and searching!

## Elasticsearch example ##

Here is a config example for Elasticsearch. To use it put the two data files
into the `${ES_HOME}/config/analysis` directory of your ES node and add
the following settings to your index. After that you can use the
`german_decompound` analyzer in your mapping.

```json
"settings": {
  "analysis": {
     "filter": {
        "german_decompounder": {
           "type": "hyphenation_decompounder",
           "word_list_path": "analysis/dictionary-de.txt",
           "hyphenation_patterns_path": "analysis/de_DR.xml",
           "only_longest_match": true,
           "min_subword_size": 4
        },
        "german_stemmer": {
           "type": "stemmer",
           "language": "light_german"
        }
     },
     "analyzer": {
        "german_decompound": {
           "type": "custom",
           "tokenizer": "standard",
           "filter": [
              "lowercase",
              "german_decompounder",
              "german_normalization",
              "german_stemmer"
           ]
        }
     }
  }
}
```

*Important:* Use the analyzer for both indexing and searching!

## Help Out! ##

If you have suggestions for improving the German dictionary, please send
a pull request, thanks! Be sure to only send "plain words", no compounds!

## License ##

See [NOTICE.txt](NOTICE.txt) for more information!
