```coffee
1.
context: Sistema::altaExemplar(nomEx: String, sexeEx: TipusSexe, nomEsp: String) : Exemplar
	pre:
		Especie.allInstances()->exists(e |
			e.nom = nomEsp and
			e.zona->exists(z |
				z.especie->size() > 2
			)
		)

	post:
		Exemplar.allInstances()->exists(ex |
			ex.oclIsNew() and
			ex.nom = nomEx and
			ex.sexe = sexeEx and
			ex.especie.nom = nomEsp and
			result = ex
		)
2.
context: Sistema::altaZona(nomZ: String) : Zona
	post:
		Zona.allInstances()->exists(z |
			z.oclIsNew() and
			z.nom = nomZ and
			result = z
		)

context: Sistema::afegirEspecieZona(nomEsp: String, zo: Zona, nomPais: Pais, estatCons: TipusEstat)
	pre:
		if not Especie.allInstances()->exists(e |
				e.nom = nomEsp
			)
		then
			nomPais.notEmpty() and
			estatCons.notEmpty() and
			Pais.allInstances()->exists(p |
				p.nom = nomPais
			)
		endif

	post:
		if not Especie.allInstances()@pre->exists(e |
				e.nom = nomEsp
			)
		then
			zo.especie->exists(e |
				e.oclIsNew() and
				e.nom = nomEsp and
				e.pais->exists(p |
					p.nom = nomPais
				) and
				Localitzacio.allInstances()->exists(l |
					l.especie.nom = nomEsp and
					l.pais.nom = nomPais and
					l.estatConservacio = estatCons
				)
			)
		else
			zo.especie->exists(e |
				e.nom = nomEsp
			)
		endif
3.
context: Sistema::consultaAgressionsRellevants(nomEx: String) : Set(TupleType(data: Data, nomFerit: String))
	pre:
		Exemplar.allInstances()->exists(ex |
			ex.nom = nomEx and
			not Localitzacio.allInstances()->exists(l |
				ex.especie.pais.nom->includes(l.pais.nom) and
				l.especie.nom = ex.especie.nom and
				l.estatConservacio = TipusEstat.EnPerillGreu
			)
		)

	body:
		result = Agressio.allInstances()->select(a |
			a.agressor.nom = nomEx and
			Localitzacio.allInstances()->exists(l |
				a.agredit.especie.pais.nom->includes(l.pais.nom) and
				l.especie.nom = a.agredit.especie.nom and
				l.estatConservacio = TipusEstat.EnPerillGreu
			)
		)->collect(a |
			Tuple{ data = a.data.data, nomFerit = a.ferit.nom }
		)
```
