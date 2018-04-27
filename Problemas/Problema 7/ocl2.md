```java
1.
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

context: Sistema::classificaConcurs(parelles: Set(TupleType(nomHome: String, nomDona: String), dorsal: Natural), concurs: Concurs) : Concurs
	pre:
		Parella.home.nom->includesAll(parelles.nomHome)
		Parella.dona.nom->includesAll(parelles.nomDona)

	post:
		ParellaParticipant.allInstances()->exists(p |
			p.home.nom = parelles.nomHome
			p.dona.nom = parelles.nomDona
			p.Activitat.oclAsType(Concurs) = concurs
		)

2.
context: Sistema::enregistrarParellesGuanyadores(parelles: Set(nomHome: String, nomDona: String, nomRecinte: String, data: Data, dorsal: Int)) : Set(ParellaGuanyadora)
	pre:
		Parella.home.nom->includesAll(parelles.nomHome)
		Parella.dona.nom->includesAll(parelles.nomDona)
		Recinte.nom->includesAll(parelles.nomRecinte)
		Data.data->includesAll(parelles.data)

		parelles->forAll(
			ParellaParticipant.allInstances()->exists(p |
				p.parella.home.nom = nomHome
				p.parella.dona.nom = nomDona
				p.activitat.recinte.nom = nomRecinte
				p.activitat.data.data = data
				p.dorsal = dorsal
			)
		)

	post:
		parelles->forAll(
			ParellaParticipant.allInstances()->select(p |
				p.parella.home.nom = nomHome
				p.parella.dona.nom = nomDona
				p.activitat.recinte.nom = nomRecinte
				p.activitat.data.data = data
				p.dorsal = dorsal
			).oclIsTypeOf(ParellaGuanyadora)
		)

context: Sistema::atributsDerivats(parella: ParellaGuanyadora) : ParellaGuanyadora
	pre:
		let concursosGuanyatsHome: Int = parella.parella.home.concursosGuanyats
		let concursosGuanyatsDona: Int = parella.parella.dona.concursosGuanyats
		let anyConcurs: Data = parella.activitat.data.year

	post:
		parella.parella.home.concursosGuanyats = concursosGuanyatsHome + 1
		parella.parella.dona.concursosGuanyats = concursosGuanyatsDona + 1
		Exhibicio->exists(e |
			e.data.year = anyConcurs + 1
			e.persona->includes(parella.parella.home)
			e.persona->includes(parella.parella.dona)
		)

3.
context: Sistema::concursosRecinte(nomRecinte: String) : Set(TupleType(data : Data, participants: Int))
	pre:
		Recinte.allInstances()->exists(r |
			r.nom = nomRecinte
		)

	body:
		Recinte.allInstances()->select(r |
			r.nom = nomRecinte
		).activitat.oclAsType(Concurs)->select(a |
			a.parellaParticipant->size() * 2 > 50 and
			not (Activitat.oclAsType(Exhibicio)->exists(e| e.data = a.data))
		)->collect(c |
			Tuple{ data = c.data, participants = c.parellaparticipant->size() * 2 }
		)
```
