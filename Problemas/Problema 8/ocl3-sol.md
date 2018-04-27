```coffee
1.
context: Sistema::altaExemplar(nomEx: String, sexeEx: TipusSexe, nomEsp: String, min: Int, max: Int, fertil: Bool)
	pre:
		Especie.allInstances()->exists(e |
			e.nom = nomEsp and
			e.zona->exists(z |
				z.especie->size() >= 2
			)
		)

	post:
		Exemplar.allInstances()->exists(ex |
			ex.oclIsNew() and
			ex.nom = nomEx and
			ex.sexe = sexeEx and
			ex.especie.nom = nomEsp and
			result = ex and
			if ex.especie.nom = "Lleó" then
				ex.oclIsTypeOf(ExemplarLleó) and
				ex.oclAsType(ExemplarLleó).minimKg = min and
				ex.oclAsType(ExemplarLleó).maximKg = max and
				if ex.sexe = TipusSexe.Femení then
					ex.oclIsTypeOf(ExemplarLleóFemella) and
					ex.oclAsType(ExemplarLleóFemella).fertil = fertil
				endif
			endif
		)
```

```coffee
2.
context: Sistema::altaZona(nomZ: String) : Zona
	post:
		Zona.allInstances()->exists(z |
			z.oclIsNew() and
			z.nom = nomZ and
			result = z
		)

context: Sistema::afegirEspecieZona(nomEsp: String, zo: Zona) : Especie
	post:
		if not Especie.allInstances()@pre->exists(e |
				e.nom = nomEsp
			)
		then
			Especie.allInstances()->exists(e |
				e.oclIsNew() and
				e.nom = nomEsp and
				result = e
			)
		endif and
		zo.especie.nom->includes(nomEsp)

context: Sistema::afegirLocalitzacio(e: Especie, nomPais: String, estatCons: TipusEstat)
	pre:
		Pais.allInstances()->exists(p |
			p.nom = nomPais
		)
	post:
		Localitzacio.allInstances()->exists(l |
			l.oclIsNew() and
			l.estatConservacio = estatConst
			l.pais.nom = nomPais and
			l.especie = e
		)
```

```coffee
3.
context: Sistema::consultaAgressionsRellevants(nomEx: String) : Set(TupleType(data: Data, ferits: Set(String)))
	pre:
		Exemplar.allInstances()->exists(ex |
			ex.nom = nomEx and
			ex.especie.localitzacio->forAll(l |
				l.estatConservacio = "SensePerill"
			)
		)

	body:
		let agressions : Set(Agressio) = Agressio.allInstances()->select(a |
			a.agressor.nom = nomEx and
			a.agredit.especie.localitzacio->exists(l |
				l.estatConservacio = "EnPerillGreu"
			)
		)
		in
			result = agressions->collect(a |
				Tuple{ data = a.data.data, ferits = a.ferit.nom }
			)
```
