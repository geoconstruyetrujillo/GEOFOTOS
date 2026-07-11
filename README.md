import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/punto_form_provider.dart'; // Tu manejador de estado
import '../widgets/foto_card.dart';             // El widget de la tarjeta de foto

class PuntoFormScreen extends StatefulWidget {
  const PuntoFormScreen({Key? key}) : super(key: key);

  @override
  State<PuntoFormScreen> createState() => _PuntoFormScreenState();
}

class _PuntoFormScreenState extends State<PuntoFormScreen> {
  @override
  Widget build(BuildContext context) {
    final formProvider = Provider.of<PuntoFormProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: const Text('📍 Registro de Punto Geodésico'),
        backgroundColor: Colors.blueGrey[900],
        foregroundColor: Colors.white,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // SECCIÓN: Datos Técnicos del Hito
            const Text('DATOS DEL HITO', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16)),
            const SizedBox(height: 10),
            TextFormField(
              decoration: const InputDecoration(
                labelText: 'Código del Punto',
                border: OutlineInputBorder(),
                hintText: 'Ej: PF-01',
              ),
              onChanged: (val) => formProvider.codigoPunto = val,
            ),
            const SizedBox(height: 15),
            Row(
              children: [
                const Text('Orden:', style: TextStyle(fontWeight: FontWeight.bold)),
                Radio(value: 'A', groupValue: formProvider.ordenPunto, onChanged: (val) => setState(() => formProvider.ordenPunto = val.toString())),
                const Text('A'),
                Radio(value: 'B', groupValue: formProvider.ordenPunto, onChanged: (val) => setState(() => formProvider.ordenPunto = val.toString())),
                const Text('B'),
                Radio(value: 'C', groupValue: formProvider.ordenPunto, onChanged: (val) => setState(() => formProvider.ordenPunto = val.toString())),
                const Text('C'),
              ],
            ),
            const SizedBox(height: 10),
            TextFormField(
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
              decoration: const InputDecoration(
                labelText: 'Altura de Antena (m)',
                border: OutlineInputBorder(),
                suffixText: 'metros',
              ),
              onChanged: (val) => formProvider.alturaAntena = double.tryParse(val) ?? 0.0,
            ),
            
            const SizedBox(height: 15),
            // Info de telemetría de fondo
            Container(
              padding: const EdgeInsets.all(12),
              color: Colors.blueGrey[50],
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceAround,
                children: [
                  Text('⏱️ Hora: ${formProvider.horaInicio ?? "Pendiente"}', style: const TextStyle(fontSize: 12)),
                  Text('📍 GPS: ${formProvider.latGps?.toStringAsFixed(4) ?? "0.0"}, ${formProvider.lngGps?.toStringAsFixed(4) ?? "0.0"}', style: const TextStyle(fontSize: 12)),
                ],
              ),
            ),

            const Padding(
              padding: EdgeInsets.symmetric(vertical: 20),
              child: Divider(),
            ),

            // SECCIÓN: Check-list de fotos obligatorias
            const Text('📸 FOTOGRAFÍAS DE CAMPO OBLIGATORIAS', style: TextStyle(fontWeight: FontWeight.bold, fontSize: 15, color: Colors.blueGrey)),
            const SizedBox(height: 10),
            
            FotoCard(
              titulo: '1. Vista General / Entorno',
              tipo: 'entorno',
              rutaImagen: formProvider.fotos['entorno'],
              onTapCamara: () => _simularCapturaCamara(context, 'entorno'),
            ),
            FotoCard(
              titulo: '2. Detalle del Hito / Placa',
              tipo: 'hito',
              rutaImagen: formProvider.fotos['hito'],
              onTapCamara: () => _simularCapturaCamara(context, 'hito'),
            ),
            FotoCard(
              titulo: '3. Equipo GNSS Instalado',
              tipo: 'equipo',
              rutaImagen: formProvider.fotos['equipo'],
              onTapCamara: () => _simularCapturaCamara(context, 'equipo'),
            ),
            FotoCard(
              titulo: '4. Medición Altura de Antena',
              tipo: 'altura',
              rutaImagen: formProvider.fotos['altura'],
              onTapCamara: () => _simularCapturaCamara(context, 'altura'),
            ),
            FotoCard(
              titulo: '5. Operador / Técnico en Campo',
              tipo: 'personal',
              rutaImagen: formProvider.fotos['personal'],
              onTapCamara: () => _simularCapturaCamara(context, 'personal'),
            ),

            const SizedBox(height: 30),

            // BOTÓN INTEGRAL DE SEGURIDAD
            SizedBox(
              width: double.infinity,
              height: 50,
              child: ElevatedButton(
                style: ElevatedButton.styleFrom(
                  backgroundColor: formProvider.esFormularioValido ? Colors.green[700] : Colors.grey[400],
                ),
                onPressed: formProvider.esFormularioValido 
                    ? () async {
                        bool guardado = await formProvider.guardarPuntoEnLocal();
                        if (guardado) {
                          ScaffoldMessenger.of(context).showSnackBar(
                            const SnackBar(content: Text('🎉 Registro guardado localmente con éxito.'))
                          );
                        }
                      }
                    : null, // Si es nulo, Flutter congela y deshabilita el botón automáticamente
                child: const Text('GUARDAR REGISTRO DE PUNTO', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
              ),
            ),
          ],
        ),
      ),
    );
  }

  // Simulación temporal de la cámara para pruebas de la UI
  void _simularCapturaCamara(BuildContext context, String tipo) {
    final provider = Provider.of<PuntoFormProvider>(context, listen: false);
    // Agrega una ruta ficticia solo para cambiar el estado visual de ❌ a ✅
    provider.guardarRutaFoto(tipo, '/storage/emulated/0/Android/data/app/$tipo.jpg');
  }
}
