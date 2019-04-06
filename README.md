# Docker SMTP Server with DKIM

#### Target of this docker project get easiest way to add smtp to any site in 5 minute.


1. Generate DKIM online https://dkimcore.org/tools/
2. Save privatekey.txt (Private key), publickey.txt (Raw format) and tinydns.txt (Tinydns Format)
to folder "dkim".
3. Replace in tinydns.txt  
    ```
    1533037334.example._domainkey.example.ru.
    ```  
    to  
    ```
    mail._domainkey.example.ru.
    ```  
    Remove from tinydns.txt all line break, quotes, commas, :3600:: if exist, white spaces.   
    Must be like this:  
    ```
    mail._domainkey.example.ru:v=DKIM1;p=MIGfMA0GCSqGSIb3DQEBxQUAA4GNADCBiQKBgQCRxzp5nT2S5xqYNPMXaHzx9FZdO+QKiZse6tOTcDeZbRzR9I/oEzMgbuDoWwQ2SsCfvlx7lzxKjDaKkbl3rxnSF1wpSre7AMqM9nZq7b5kX+YzWXzuTiwCMBl6bbnAi/x+qePV9lURJVu5YcblYYOAqWZ/3F/8DDRFGeBjDwcwIDAQAB
    ```
      
4. Add TXT record to your domain in DNS zone    
    
    KEY:   
    ```
    mail._domainkey
    ```   
    
    VALUE:   
    ```
    v=DKIM1;p=MIGfMA0GCSqGSIb3DQEBxQUAA4GNADCBiQKBgQCRxzp5nT2S5xqYNPMXaHzx9FZdO+QKiZse6tOTcDeZbRzR9I/oEzMgbuDoWwQ2SsCfvlx7lzxKjDaKkbl3rxnSF1wpSre7AMqM9nZq7b5kX+YzWXzuTiwCMBl6bbnAi/x+qePV9lURJVu5YcblYYOAqWZ/3F/8DDRFGeBjDwcwIDAQAB
    ```   
    
5. Delete key from https://dkimcore.org/tools/



Build image example:
```
cd smtp && docker build -t smtp .
```

Docker compose example:

```
version: "3"

services:
  site:
    image: site:latest
    depends_on:
      - smtp
    links:
      - smtp
    volumes:
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
  smtp:
    image: smtp:latest
    environment:
      MAILNAME: "site"
    volumes:
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
```

Usage example in golang:
```
package main

import (
	"bytes"
	"log"
	"net/smtp"
)

func main() {
	
	// Connect to the remote SMTP server.
	c, err := smtp.Dial("smtp:25")
	if err != nil {
		log.Fatal(err)
	}
	defer c.Close()
	
	// Set the sender and recipient.
	c.Mail("sender@example.org")
	c.Rcpt("recipient@example.net")
	
	// Send the email body.
	wc, err := c.Data()
	if err != nil {
		log.Fatal(err)
	}
	defer wc.Close()
	buf := bytes.NewBufferString("This is the email body.")
	if _, err = buf.WriteTo(wc); err != nil {
		log.Fatal(err)
	}
}
```

