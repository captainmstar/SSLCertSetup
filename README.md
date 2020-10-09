# SSL Certificate Setup

### Below are steps that can be used to create the necessary SSL certificates needed for establishing 2 way/mutual TLS socket communication. Also included are the steps to create jks files to be used on the Super Computer for implementing a Java based socket connection.

	
## Create Root Certificate Authority
- ### Create Root Key
        openssl genrsa -des3 -out rootCA.key 4096
        If you want a non password protected key just remove the -des3 option. **This is what I did.
			
- ### Create and self sign the Root Certificate
		openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt
	
	
## IVI - Create signed certificate
- ### Create certificate key
        openssl genrsa -out IVI.key 2048
			
- ### Create certificate signing request with configuration file (ivi.conf) which sets SANs, etc
		openssl req -new -sha256 -key IVI.key -config ivi.conf -out IVI.csr
			
- ### Create certificate with CA, CSR
		openssl x509 -req -in IVI.csr -CA rootCA.crt -CAkey CA.key -CAcreateserial -extfile ivi.ext -out IVI.crt -days 500 -sha256
	
## SC - Create signed certificate
- ### Create certificate key
		openssl genrsa -out SC.key 2048
		
- ### Create certificate signing request with configuration file (sc.conf) which sets SANs, etc
		openssl req -new -sha256 -key SC.key -config sc.conf -out SC.csr
		
- ### Create certificate with CA, CSR
		openssl x509 -req -in SC.csr -CA rootCA.crt -CAkey CA.key -CAcreateserial -extfile sc.ext -out SC.crt -days 500 -sha256
	
## SC - Convert SC crt to jks format as needed by Java
- ### Cert(crt) to jks - Only need for SC because I can export PEM from Portecle - Both commands below required pwd creation
- ### export crt to p12 file
		openssl pkcs12 -export -in SC.crt -inkey SC.key -out SC.p12
				
- ### convert p12 to jks
		keytool -importkeystore -srckeystore SC.p12 -srcstoretype PKCS12 -destkeystore SC.jks -deststoretype JKS
		
## SC - Add root CA to truststore of SC
- ### create a new truststore and import root crt to truststore
		keytool -keystore truststore.jks -importcert -file rootCA.crt -alias ca-root -storepass secret
		
## Misc
- ### Verify crt file
		openssl x509 -in IVI.crt -text -noout
		
- ### Verify csr file
	    openssl req -in IVI.csr -noout -text
