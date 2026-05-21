@startuml abrirEspacios-analisis
skinparam linetype polyline

rectangle #CDEBA5 ":Universidad Disponible" as UniversidadDisponible
package abrirEspacios as "abrirEspacios()" {
    rectangle #629EF9 EspaciosView
    rectangle #b5bd68 EspacioController
    rectangle #F2AC4E EspacioRepository
    rectangle #F2AC4E Espacio
}

rectangle #CDEBA5 ":Universidad Disponible" as UniversidadDisponibleSalida
rectangle #CDEBA5 ":collaboration crearEspacio" as CrearEspacio
rectangle #CDEBA5 ":collaboration editarEspacio" as EditarEspacio
rectangle #CDEBA5 ":collaboration eliminarEspacio" as EliminarEspacio

UniversidadDisponible -r-> EspaciosView: abrirEspacios()

EspaciosView -d-> EspacioController: obtenerEspacios() : List<Espacio>

EspacioController --> EspacioRepository: findAll() : List<Espacio>

EspacioRepository -- Espacio

EspaciosView -d-> EspacioController: filtrarEspacios(criterio) : List<Espacio>

EspacioController --> EspacioRepository: findByCriterio(criterio) : List<Espacio>

EspaciosView ..> UniversidadDisponibleSalida: completarGestion()
EspaciosView ..> CrearEspacio: crearEspacio()
EspaciosView ..> EditarEspacio: editarEspacio()
EspaciosView ..> EliminarEspacio: eliminarEspacio()

@enduml
