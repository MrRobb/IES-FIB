
```java
context: Sistema::afegirTasca(nomProjecte: String,
                              codiTasca: String,
                              pressupostTasca: Int) : Tasca
	pre:
		Projecte.allInstances()->exists(p |
			p.nom = nomProjecte and
			p.pressupost >= pressupostTasca + Tasca.allInstances()->sum()
		)

	post:
		Tasca.allInstances()->exists(t |
			t.oclIsNew() and
			t.codiTasca = codiTasca and
			t.pressupost = pressupostTasca
			result = t
		)

context: Sistema::crearAssignacioBasica(em: Empleat,
                                        pr: Projecte,
                                        dataInici: Date,
                                        dataFi: Date,
                                        complementSou: Int,
                                        ta: Tasca,
                                        horesDedicacio: Int) : AssignacioBasica
	pre:
		AssigacioEspecial.allInstances()->select(a |
			a.Empleat.codi = em.codi
		)->size() = 0

	post:
		AssignacioBasica.allInstances()->exists(a |
			a.oclIsNew() and
			a.Empleat.codi = em.codi and
			a.Projecte.nom = pr.nom and
			a.dataInici = dataInici and
			a.dataFi = dataFi and
			a.complementSou = complementSou and
			a.Tasca.codiTasca = ta.codiTasca
			a.Realitzacio.horesDedicacio = horesDedicacio
			result = a
		)

context: Sistema::consultaProjectesBenComplementats() : Set(TupleType(nomProj: String, codis: Set(Integer)))
	pre:

	post:
		let projectes : Set(Projecte) =
		Projecte.allInstances()->select(p |
			p.Assignacio.oclIsType(AssignacioBasica).oclAsType(AssignacioBasica).complementSou > 1000 and
			Realitzacio.allInstances()->select(r |
				r.AssignacioBasica.Projecte.nom = p.nom
			)->size() > 3 and
			projectes->collect(
				Tuple{ nomProj = p.nom, codis = p.Tasca.allInstances().codiTasca }
			)
		)
		result = projectes		
```
