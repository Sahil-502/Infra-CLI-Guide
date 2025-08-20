### •	Create Private SSL Key.

pic

![Alt text](D:\"1. GIT"\Infra-CLI-Guide\SSL\PIC\ssl5-cmd-1)


### •	Create a public SSL on Zero SSL and paste it:

•	Add CNAME in your DNS and verify the certificate.
•	Download the certificate Zip file:


pic

•	Open the folder and install certificate.crt file.

pic

•	Now you have a Personal Information Exchange (.pfx) file.
o	Extract the Certificate and Key from the .pfx File.
o	wsl –install (Optional)
o	apt install openssl  (If you don’t have)
o	openssl pkcs12 -in yourfile.pfx -nocerts -out key.pem -nodes
o	openssl pkcs12 -in yourfile.pfx -clcerts -nokeys -out cert.pem





•	Now you have cert.pem and key.pem file.
•	Upload it in GCP.
Verify the Chain
•	Open cert.pen file and ca_bundle.crt file in notepad. Merge it and upload.
