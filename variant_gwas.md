# Variant GWAS

## Parameters

* `ep` Endpoint
  * default: https://togovar-dev.biosciencedbc.jp/sparql
* `tgv_id` TogoVar ID
  * default: tgv47264307

## Endpoint

{{ ep }}

## `tgv2rs`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX dct: <http://purl.org/dc/terms/> 
SELECT ?rsid 
FROM <http://togovar.biosciencedbc.jp/variation>

WHERE {
       VALUES ?variant { "{{tgv_id}}" }.
       ?s dct:identifier ?variant;
         rdfs:seeAlso ?dbsnp.
  
       FILTER STRSTARTS(STR(?dbsnp), 'http://identifiers.org/dbsnp/').
       BIND (REPLACE(STR(?dbsnp),'http://identifiers.org/dbsnp/','') as ?rsid).
}
```

## `rsids`
```javascript
({tgv2rs}) => {
  return tgv2rs.results.bindings.map(x => '"' + x.rsid.value + '"').join(" ");
}
```

## `rs2gwas`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX terms: <http://med2rdf.org/gwascatalog/terms/> 
PREFIX gwas: <http://rdf.ebi.ac.uk/terms/gwas/> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX dct: <http://purl.org/dc/terms/> 

SELECT ?rs_id ?variant_and_risk_allele ?raf ?p_value ?odds_ratio ?ci_text ?beta ?mapped_trait ?mapped_trait_uri  ?pubmed ?pubmed_id ?study ?study_id ?initial_sample_size ?replication_sample_size  
FROM  <http://togovar.biosciencedbc.jp/gwas-catalog>
WHERE {
    VALUES ?rs_id { {{rsids}} }.
  
    ?assoc a gwas:Association ;
      terms:study ?study;  #  study:GCST010572 ;
      terms:snps ?rs_id;  #  "rs671" ;  
      terms:strongest_snp_risk_allele ?variant_and_risk_allele;
      terms:risk_allele_frequency ?raf;
      terms:p_value ?p_value;  # 2E-70
      terms:odds_ratio ?odds_ratio;   #0.22 
      terms:beta ?beta;  #  "NA" ;
      terms:ci_text ?ci_text;  # "[0.2-0.24] unit increase" ;
      terms:mapped_trait ?mapped_trait;   #"sweet liking measurement" ;
      terms:mapped_trait_uri ?mapped_trait_uri ;  # <http://www.ebi.ac.uk/efo/EFO_0010156> ;
      dct:references ?pubmed;   #pubmed:32572145 ;
      gwas:has_pubmed_id ?pubmed_id.  # "32572145" .

    ?study dct:identifier ?study_id;  #"GCST010426" ;
      terms:initial_sample_size ?initial_sample_size;  # "27,617 European ancestry individuals with some college education, 20,253 European ancestry individuals without college education, 8,128 African ancestry individuals with some college education, 8,537 African ancestry individuals without college education, 1,138 Asian ancestry individuals with some college education, 10,214 Asian ancestry individuals without college education, 1,869 Hispanic/Latin American individuals with some college education, 3,276 Hispanic/Latin American individuals without college education"@en ;
      terms:replication_sample_size ?replication_sample_size.  # "131,584 European ancestry individuals with some college education, 110,940 European ancestry individuals without college education, 1,494 African ancestry individuals with some college education, 5,704 African ancestry individuals without college education, 2,339 Asian ancestry individuals with some college education, 8,567 Asian ancestry individuals without college education, 4,133 Hispanic individuals with some college education, 7,498 Hispanic individuals without college education"@en ;
  }
```
## `result`
```javascript

({
  json({rs2gwas}){
    let data = [];
    rs2gwas.results.bindings.map(x => {
      let obj={
        rs_id : x.rs_id.value,
        variant_and_risk_allele : x.variant_and_risk_allele.value,
        raf: x.raf.value,
        p_value : x.p_value.value,
        odds_ratio : x.odds_ratio.value,
        ci_text : x.ci_text.value,
        beta : x.beta.value,
        mapped_trait : x.mapped_trait.value,
        mapped_trait_uri : x.mapped_trait_uri.value,
        pubmed : x.pubmed.value,
        pubmed_id : x.pubmed_id.value,
        study : x.study.value,
        study_detail : x.study_id.value,
        initial_sample_size : x.initial_sample_size.value,
        replication_sample_size : x.replication_sample_size.value
      };
      data.push(obj);
    });
    return data;
  }
})
```