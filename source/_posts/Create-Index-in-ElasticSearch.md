title: Create Index in ElasticSearch
date: 2015-04-11 09:03:33
categories: dev
tags: [ElasticSearch]
---

## Create Index

```shell
curl -XPUT '10.0.0.101:9200/truesight/' -d '{
  "settings" : {
    "number_of_shards" : 10,
    "number_of_replicas": 1
  }
  }'
```

## Create Type

index has 3 types:
  1. analyzed: fulltext index
  2. not_analyzed: whole value index, not fulltext index
  3. no: no index

```shell
curl -XPUT '10.0.0.101:9200/truesight' -d '{
  "settings" : {
    "number_of_shards" : 10,
    "index.number_of_replicas": 1
    },
    "mappings" : {
      "page" : {
        "_ttl": {
          "enabled": "true",
          "default": "7d"
          },
          "properties" : {
            "x_location":  { "type" : "string", "index" : "not_analyzed" },
            "x_record_type":  { "type" : "string", "index" : "not_analyzed" },
            "x_page_id":  { "type" : "long", "index" : "not_analyzed" },
            "x_session_id":  { "type" : "long", "index" : "not_analyzed" },
            "x_server_id":  { "type" : "string", "index" : "analyzed" },
            "x_page_name":  { "type" : "string", "index" : "analyzed" },
            "cs_host":  { "type" : "string", "index" : "analyzed" },
            "cs_uri_stem":  { "type" : "string", "index" : "analyzed" },
            "cs_uri_query":  { "type" : "string", "index" : "analyzed" },
            "cs_referrer":  { "type" : "string", "index" : "analyzed" },
            "x_redirect_host":  { "type" : "string", "index" : "analyzed" },
            "x_redirect_url":  { "type" : "string", "index" : "analyzed" },
            "cs_cookie":  { "type" : "string", "index" : "analyzed" },
            "sc_set_cookie":  { "type" : "string", "index" : "analyzed" },
            "x_cs_post":  { "type" : "string", "index" : "analyzed" },
            "x_start_time":  { "type" : "date", "index" : "not_analyzed" },
            "x_start_time_utc":  { "type" : "long", "index" : "not_analyzed" },
            "x_end_time":  { "type" : "date", "index" : "not_analyzed" },
            "x_end_time_utc":  { "type" : "long", "index" : "not_analyzed" },
            "c_ip":  { "type" : "string", "index" : "analyzed" },
            "c_port":  { "type" : "integer", "index" : "not_analyzed" },
            "x_first_public_ip":  { "type" : "string", "index" : "analyzed" },
            "s_ip":  { "type" : "string", "index" : "analyzed" },
            "s_port":  { "type" : "integer", "index" : "not_analyzed" },
            "sc_bytes":  { "type" : "integer", "index" : "not_analyzed" },
            "x_throughput":  { "type" : "integer", "index" : "no" },
            "x_tcp_packet_count":  { "type" : "integer", "index" : "no" },
            "x_tcp_rtt_count":  { "type" : "integer", "index" : "no" },
            "x_tcp_rtt":  { "type" : "integer", "index" : "no" },
            "x_tcp_ooo":  { "type" : "integer", "index" : "no" },
            "x_tcp_retrx":  { "type" : "integer", "index" : "no" },
            "x_redirect_time":  { "type" : "integer", "index" : "no" },
            "x_redirect_ssl_time":  { "type" : "integer", "index" : "no" },
            "x_redirect_ssl_count":  { "type" : "string", "index" : "no" },
            "x_redirect_process_time":  { "type" : "integer", "index" : "no" },
            "x_redirect_network_time":  { "type" : "integer", "index" : "no" },
            "x_redirect_count":  { "type" : "integer", "index" : "no" },
            "x_ssl_time":  { "type" : "integer", "index" : "no" },
            "x_ssl_count":  { "type" : "integer", "index" : "no" },
            "x_process_time":  { "type" : "integer", "index" : "no" },
            "x_network_time":  { "type" : "integer", "index" : "no" },
            "x_idle_time":  { "type" : "integer", "index" : "no" },
            "x_e2e_time":  { "type" : "integer", "index" : "no" },
            "cs_method":  { "type" : "string", "index" : "no" },
            "cs_version":  { "type" : "string", "index" : "no" },
            "x_sc_mimetype":  { "type" : "string", "index" : "no" },
            "sc_status":  { "type" : "integer", "index" : "no" },
            "x_document":  { "type" : "boolean", "index" : "no" },
            "x_container":  { "type" : "boolean", "index" : "no" },
            "x_aborted":  { "type" : "boolean", "index" : "no" },
            "x_secure":  { "type" : "boolean", "index" : "no" },
            "x_secure_count":  { "type" : "integer", "index" : "no" },
            "x_content_count":  { "type" : "integer", "index" : "no" },
            "x_errored_count":  { "type" : "integer", "index" : "no" },
            "x_slt_broken":  { "type" : "boolean", "index" : "no" },
            "x_nw_error_count":  { "type" : "integer", "index" : "no" },
            "x_cl_error_count":  { "type" : "integer", "index" : "no" },
            "x_sv_error_count":  { "type" : "integer", "index" : "no" },
            "x_ap_error_count":  { "type" : "integer", "index" : "no" },
            "x_ct_error_count":  { "type" : "integer", "index" : "no" },
            "x_cu_error_count":  { "type" : "integer", "index" : "no" },
            "x_nw_info_count":  { "type" : "integer", "index" : "no" },
            "x_cl_info_count":  { "type" : "integer", "index" : "no" },
            "x_sv_info_count":  { "type" : "integer", "index" : "no" },
            "x_ap_info_count":  { "type" : "integer", "index" : "no" },
            "x_ct_info_count":  { "type" : "integer", "index" : "no" },
            "x_cu_info_count":  { "type" : "integer", "index" : "no" },
            "x_errors":  { "type" : "string", "index" : "no" },
            "x_info":  { "type" : "string", "index" : "no" },
            "x_wp":  { "type" : "string", "index" : "no" },
            "cs_user_agent":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_country":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_country_string":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_region":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_region_string":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_city":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_metro_area":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_organization":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_isp":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_dns_name":  { "type" : "string", "index" : "analyzed" },
            "x_first_public_geo_latitude":  { "type" : "double", "index" : "no" },
            "x_first_public_geo_longitude":  { "type" : "double", "index" : "no" },
            "x_page_render_time":  { "type" : "integer", "index" : "analyzed" },
            "x_prm_instrumented":  { "type" : "string", "index" : "no" },
            "x_aborted_count":  { "type" : "integer", "index" : "no" },
            "x_errored_aborted_count":  { "type" : "integer", "index" : "no" },
            "x_akamai_download_time_sum":  { "type" : "integer", "index" : "no" },
            "x_delivery_mode":  { "type" : "string", "index" : "no" },
            "x_object_delivery_origin_count":  { "type" : "integer", "index" : "no" },
            "x_object_delivery_akacachemgt_count":  { "type" : "integer", "index" : "no" },
            "x_object_delivery_akacachehit_count":  { "type" : "integer", "index" : "no" },
            "x_object_delivery_akacachemiss_count":  { "type" : "integer", "index" : "no" },
            "collectorfeednames":  { "type" : "string", "index" : "no" },
            "browser":  { "type" : "string", "index" : "analyzed" },
            "os":  { "type" : "string", "index" : "analyzed" },
            "trace":  { "type" : "string", "index" : "no" },
            "x_application_name":  { "type" : "string", "index" : "no" },
            "x_custom_acceptlanguage":  { "type" : "string", "index" : "no" },
            "x_custom_continent":  { "type" : "string", "index" : "no" },
            "x_custom_referrer_name":  { "type" : "string", "index" : "no" }
          }
        }
      }
    }'

```

-EOF-
Tao Wu@CA, USA
