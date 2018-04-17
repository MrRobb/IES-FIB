```java
context: Sistema::creaNovaActivitat(nomRecinte: String, data: Date) : Activitat
	pre:
		Recinte.allInstances()->exists(r |
			r.nom = nomRecinte
		) and
		Data.allInstances()->exists(d |
			d.data = data
		)
	post:
		Activitat.allInstances()->exists(a |
			a.oclIsNew() and
			a.nom = nomRecinte and
			a.data = data and
			result = a
		)

context: Sistema::classificaConcurs(parelles: Set(TupleType(nomHome: String, nomDona: String, dorsal: Natural))) : Concurs
	pre:
		Parella.allInstances().home.nom->includesAll(parelles.nomHome)
		Parella.allInstances().dona.nom->includesAll(parelles.nomDona)

context: Sistema::classificaExhibició() : Exhibicio
```
