# Haproxy-freeipa-conf
Make freeipa hosts available behind haproxy load balancer
```
...
frontend main 
    bind :80
    redirect scheme https code 301 if !{ ssl_fc }
frontend main_ssl
    bind :443 ssl crt /etc/haproxy/ssl/
    use_backend freeipa if { ssl_fc_sni freeipa.mydomain.ru  }

backend freeipa
  mode http
  balance roundrobin
  cookie SERVERID insert indirect nocache httponly secure
  
  acl hdr_req_ipa01 req.hdr(Cookie) -m sub ipa01 
  acl hdr_req_ipa02 req.hdr(Cookie) -m sub ipa02 
  acl hdr_req_ipa03 req.hdr(Cookie) -m sub ipa03 
  http-request set-header Host freeipa01.inside.mydomain.ru if hdr_req_ipa01
  http-request replace-header Referer ^https?://freeipa\.mydomain\.ru(.*)$   https://freeipa01\.inside\.mydomain\.ru\1  if hdr_req_ipa01
  http-request set-header Host freeipa02.inside.mydomain.ru if hdr_req_ipa02
  http-request replace-header Referer ^https?://freeipa\.mydomain\.ru(.*)$ https://freeipa01\.inside\.mydomain\.ru\1 if hdr_req_ipa02
  http-request set-header Host freeipa03.inside.mydomain.ru if hdr_req_ipa03
  http-request replace-header Referer ^https?://freeipa\.mydomain\.ru(.*)$ https://freeipa01\.inside\.mydomain\.ru\1 if hdr_req_ipa03
  
  acl hdr_ipa01 res.hdr(Location) -m sub freeipa01.inside.mydomain.ru
  acl hdr_ipa02 res.hdr(Location) -m sub freeipa02.inside.mydomain.ru
  acl hdr_ipa03 res.hdr(Location) -m sub freeipa03.inside.mydomain.ru
  http-response replace-header Set-Cookie ^Domain=freeipa01\.inside\.mydomain\.ru(.*) Domain=freeipa\.mydomain\.ru\1 if hdr_ipa01
  http-response replace-value Location ^https?://freeipa01\.inside\.mydomain\.ru(.*)$ https://freeipa\.mydomain\.ru\1 if hdr_ipa01
  http-response replace-header Set-Cookie ^Domain=freeipa02\.inside\.mydomain\.ru(.*) Domain=freeipa\.mydomain\.ru\1 if hdr_ipa02
  http-response replace-value Location ^https?://freeipa02\.inside\.mydomain\.ru(.*)$ https://freeipa\.mydomain\.ru\1 if hdr_ipa02
  http-response replace-header Set-Cookie ^Domain=freeipa03\.inside\.mydomain\.ru(.*) Domain=freeipa\.mydomain\.ru\1 if hdr_ipa03
  http-response replace-value Location ^https?://freeipa03\.inside\.mydomain\.ru(.*)$ https://freeipa\.mydomain\.ru\1 if hdr_ipa03
  
  server ipa01 freeipa01.inside.mydomain.ru:443 check port 443 inter 5s rise 2 fall 5 cookie ipa01 weight 9 ssl verify none
  server ipa02 freeipa02.inside.mydomain.ru:443 check port 443 inter 5s rise 2 fall 5 cookie ipa02 weight 1 ssl verify none
  server ipa03 freeipa03.inside.mydomain.ru:443 check port 443 inter 5s rise 2 fall 5 cookie ipa03 weight 3 ssl verify none
  ```
