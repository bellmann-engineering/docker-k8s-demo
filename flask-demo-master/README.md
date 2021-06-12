# Die vielleicht kleinste Webseite der Welt

Dieses auf Python-Flask basierende Demo ist zu schulungszwecken gedacht.
Die Anwendung ist sichtbar auf Port 5000 mit Docker run soll sie auf Port 8080 gemapped werden.

1. Download oder clone source
2. docker build flask-demo-master/ -t kbellmann/tinyweb:1.1
3. Test:
- docker run --name tinyweb -p 8080:5000 -e USER='Kai' kbellmann/tinyweb:1.1
- Browser: http://localhost:8080/
4. docker push [ACCOUNT]/tinyweb:1.1
