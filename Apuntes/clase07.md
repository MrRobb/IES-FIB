```java
context: Sistema::afegirTasca(nomProj: String, nTasca: Int) : TascaProj
	pre:
		Projecte.allInstances()->exists(p |
			p.nom = nomProj
		) and
		Tasca.allInstances()->exists(t |
			t.codiTasca = nTasca
		)
	post:
		Projecte.allInstances()->exists(p |
			p.nom = np
		).tasca.codiTasca->includes(nTasca)		
```

```java
context: Sistema::crearAssignacioBasica(codiEmpl: Int, nomProj: String, dataInici: Data, complementSou: Int) : AssignacioBasica
	pre:
		Empleat.allInstances()->exists(e |
			e.codi = codiEmpl and
			e.assignacio->select(a |
				a.oclAsType(AssigacioEspecial)->size() = 0
			)
		) and
		Projecte.allInstances()->exists(p |
			p.nom = nomProj
		)
	post:
		Assignacio.allInstances()->exists(a |
			a.oclIsNew() and
			a.dataInici = dataInici and
			a.empleat.codi = codiEmpl and
			a.projecte.nom = nomProj and
			a.oclIsTypeOf(AssignacioBasica) and
			a.oclAsType(AssignacioBasica).complementSou = complementSou and
			result = a
		)

context: Sistema::afegirRealizatcio(codiTasca: String, horesDed: Int, assig: AssignacioBasica)
	pre:
		Tasca.allInstances()->exists(t |
			t.codi = codiTasca
		)
	post:
		Realitzacio.allInstances()->exists(r |
			r.oclIsNew() and
			r.tasca.codi = codiTasca and
			r.assignacioBasica = assig and
			r.horesDedicacio = horesDed
		)
```

```java
context: Sistema::consultaProjectesBenComplementats() : Set(TupleType(nomProj : String, Set(codiTasca: String)))
	body:
		let projectes : Set(Projecte) =
			Projecte.allInstances()->select(p |
				p.assignacio->exists(a |
					a.oclAsType(AssignacioBasica).complementSou > 1000 and
					a.oclAsType(AssignacioBasica).realitzacio->size() >= 3
			) in projectes->collect(p |
				Tuple{ nomProj = p.nom, codis = p.tasca.codiTasca }
			)
```
