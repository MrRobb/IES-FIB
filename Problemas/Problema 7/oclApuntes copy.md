|                                     	| PRE                                	| POST                             	|
|-------------------------------------	|------------------------------------	|----------------------------------	|
| Nueva instancia de clase            	| -                                  	| OclIsNew() + Atributos           	|
| Nueva instancia de asociación       	| Comprobar que los extremos existen 	| Unirlos                          	|
| Nueva instancia de clase asociativa 	| Comprobar que los extremos existen 	| OclIsNew() + Atributos + Unirlos 	|
| Especialización                     	| Comprobar que el objeto existe     	| OclIsTypeOf(...)                 	|

> Al final del diagrama se comprueban que todas las restricciones tanto gráficas como textuales como de UML se cumplen.

```coffee
1.
context: Sistema::creaNovaActivitat(nomR: String, data: Date, esConcurs: Bool) : Activitat
	pre:
		Recinte.allInstances()->exists(r |
			r.nom = nomR
		) and
		Data.allInstances()->exists(d |
			d.data = data
		)

	post:
		Activitat.allInstances()->exists(a |
			a.oclIsNew() and
			a.recinte.nom = nomRecinte and
			a.data.data = data and
			if esConcurs then
				a.oclIsTypeOf(Concurs)
			else
				a.oclIsTypeOf(Exhibicio)
			endif and
			result = a
		)

context: Sistema::creaParellaParticipant(nomH: String, nomD: String, a: Activitat, dorsal: Int)
	pre:
		a.oclIsTypeOf(Concurs)
		Persona.allInstances()->exists(p |
			p.nom = nomH
		)
		Persona.allInstances()->exists(p |
			p.nom = nomD
		)

	post:
		if not Parella.allInstances()@pre->exists(p |
			p.home.nom = nomH and
			p.dona.nom = nomD
		)
		then
			Parella.allInstances()->exists(p |
				p.oclIsNew() and
				p.home.nom = nomH and
				p.dona.nom = nomD
			)
		endif

		ParellaParticipant.allInstances()->exists(pp |
			pp.oclIsNew() and
			pp.dorsal = dorsal and
			pp.parella.home.nom = nomH and
			pp.parella.dona.nom = nomD and
			p.Concurs = concurs
		)
```

```coffee
3.
context: Sistema::concursosRecinte(nomRecinte: String) : Set(TupleType(data : Data, participants: Int))
	pre:
		Recinte.allInstances()->exists(r |
			r.nom = nomRecinte
		)

	body:
		result = Recinte.allInstances()->select(r |
				r.nom = nomRecinte
			).activitat.oclIsTypeOf(Concurs)->select(a |
				a.parellaParticipant->size() * 2 > 50 and
				not (Exhibicio.allInstances()->exists(e| e.data = a.data))
			)->collect(c |
				Tuple{ data = c.data, participants = c.parellaparticipant->size() * 2 }
			)
```
