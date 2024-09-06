How to redirect to a different domain using Nginx?

server {
        listen 80;
        listen 443;
        server_name  .domain.example;

        return 301 $scheme://newdomain.example$request_uri;
}

server {
        listen 80;
        listen 443;
        server_name  thptdongdo.mobiedu.vn;

        return 301 $scheme://cdhtdongdops.mobiedu.vn$request_uri;
}
