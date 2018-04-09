> S'han de passar les claus primaries de les classes a les que està relacionada
>
> Les instàncies de les associacions a les que està relacionada sempre que tinguin un mínim 1.
>
> Les relacions necessàries per si fos d'una subclasse

```java
context System::altaServei(nom: String, servei: TServ) : Servei
	pre:
		Companyia.allInstances()->exists(c |
			c.nom = nom and
			c.ProgramaDeFidelitzacio->size() < 20
		)

	post:
		Servei.allInstances()->exists(s |
			s.oclIsNew() and
			s.companyia.nom = nom and
			s.tipusServei.nom = servei and
			result = s
		)
```

```java
context System::altaVol(codi: String, data: Data, servei: Servei, continent: String)
	pre:
		servei.tipusServei.nom = TServei::transoceanic implies
			Continent.allInstances()->exists(c | c.nom = continent)

	post:
		Vol.allInstances()->exists(v |
			v.oclIsNew() and
			v.codi = codi
			v.data = data and
			v.servei = servei and
			if servei.tipusServei.nom = TServei::transoceanic then
				v.oclIsType(Transoceanic).continent.nom = c
			endif
		)
```

```java
context System::altaInscripcio(dc: String, np: String, di: Data, nc: String)
	pre:
		di >= today() and
		Client.allInstances()->exists(c |
			c.dni = dc
		).vol.Servei.ProgramaDeFidelitzacio->includes(
			Companyia.allInstances()->exists(c |
				c.nom = nc
			).ProgramaDeFidelitzacio->exists(p |
				p.nom = np
			)
		)

	post:
		Inscripcio.allInstances()->exists(i |
			i.oclIsNew() and
			i.data = di and
			i.client.dni = dc and
			i.ProgramaDeFidelitzacio.nom = np and
			i.ProgramaDeFidelitzacio.Companyia.nom = nc
		)
```

```java
context System::consultaClient(nc: String) : Set(TupleType(dni: String, data: Set(Date)))
	pre:
		Companyia.allInstances()->exists(c |
			c.nom = nc
		)

	body:
		let clients: Set(Client) = Client.allInstances()->select(c |
			c.edat > 40 and
			c.Vol->select(v |
				v.Servei.Companyia.nom = nc and
				v.oclAsType(Transoceanic)->size() > 3
			) in clients->collect(c |
				Tuple{ dni = c.dni, data = c.vol.data }
			)
		)
```
