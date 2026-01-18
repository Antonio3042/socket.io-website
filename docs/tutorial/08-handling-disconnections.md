import 'package:flutter/material.dart';
import 'dart:async';
import 'dart:math';

void main() {
  runApp(const WalkieTalkieApp());
}

class WalkieTalkieApp extends StatelessWidget {
  const WalkieTalkieApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Walkie-Talkie HTML Style',
      theme: ThemeData(
        fontFamily: 'Arial', // Estilo HTML
        useMaterial3: true,
      ),
      home: const WalkieTalkieScreen(),
    );
  }
}

// =============== WALKIE-TALKIE HTML STYLE ===============
class WalkieTalkieScreen extends StatefulWidget {
  const WalkieTalkieScreen({super.key});

  @override
  State<WalkieTalkieScreen> createState() => _WalkieTalkieScreenState();
}

class _WalkieTalkieScreenState extends State<WalkieTalkieScreen> 
    with SingleTickerProviderStateMixin {
  
  // Estados do app (como variáveis JS)
  bool _isTalking = false;
  bool _isConnected = false;
  bool _isReceiving = false;
  String _statusMessage = "Ready to connect";
  String _channel = "CH-01";
  double _batteryLevel = 0.85;
  double _signalStrength = 0.7;
  List<Map<String, dynamic>> _devices = [];
  List<Map<String, dynamic>> _messages = [];
  
  // Controles de áudio
  double _volume = 0.8;
  double _squelch = 0.5;
  Timer? _talkTimer;
  Timer? _receiveTimer;
  AnimationController? _waveController;
  Random _random = Random();
  
  // Cores estilo HTML/CSS
  final Map<String, Color> _cssColors = {
    'primary': const Color(0xFF4A6FA5),
    'secondary': const Color(0xFF166088),
    'success': const Color(0xFF4CAF50),
    'danger': const Color(0xFFF44336),
    'warning': const Color(0xFFFF9800),
    'dark': const Color(0xFF212121),
    'light': const Color(0xFFF5F5F5),
    'gray': const Color(0xFF9E9E9E),
  };
  
  @override
  void initState() {
    super.initState();
    
    // Inicializar animação de ondas
    _waveController = AnimationController(
      duration: const Duration(milliseconds: 1500),
      vsync: this,
    )..repeat(reverse: true);
    
    // Simular conexão inicial
    _simulateBoot();
  }
  
  void _simulateBoot() {
    // Sequência de boot estilo sistema
    Future.delayed(const Duration(milliseconds: 500), () {
      setState(() => _statusMessage = "Initializing radio...");
    });
    
    Future.delayed(const Duration(milliseconds: 1200), () {
      setState(() => _statusMessage = "Scanning channels...");
    });
    
    Future.delayed(const Duration(milliseconds: 2000), () {
      setState(() {
        _isConnected = true;
        _statusMessage = "Connected to network";
        _devices = [
          {'id': '001', 'name': 'Alpha Team', 'signal': 0.9, 'active': true},
          {'id': '002', 'name': 'Bravo Team', 'signal': 0.7, 'active': true},
          {'id': '003', 'name': 'Charlie Team', 'signal': 0.5, 'active': false},
          {'id': '004', 'name': 'Delta Team', 'signal': 0.8, 'active': true},
        ];
        _messages = [
          {'sender': 'System', 'message': 'Network established', 'time': '09:00'},
          {'sender': 'Alpha', 'message': 'Team in position', 'time': '09:02'},
          {'sender': 'Bravo', 'message': 'All clear here', 'time': '09:05'},
        ];
      });
      
      // Simular transmissões periódicas
      _receiveTimer = Timer.periodic(const Duration(seconds: 10), (timer) {
        if (!_isTalking && _devices.isNotEmpty && _random.nextDouble() > 0.7) {
          _simulateIncomingMessage();
        }
      });
    });
  }
  
  void _simulateIncomingMessage() {
    if (_isReceiving) return;
    
    final senders = ['Alpha', 'Bravo', 'Delta'];
    final messages = [
      'Roger that',
      'Moving to position',
      'Contact, visual confirmed',
      'Requesting backup',
      'All clear',
      'Over and out'
    ];
    
    setState(() {
      _isReceiving = true;
      _messages.insert(0, {
        'sender': senders[_random.nextInt(senders.length)],
        'message': messages[_random.nextInt(messages.length)],
        'time': '${DateTime.now().hour.toString().padLeft(2, '0')}:${DateTime.now().minute.toString().padLeft(2, '0')}'
      });
      
      // Limitar histórico
      if (_messages.length > 20) {
        _messages = _messages.sublist(0, 20);
      }
    });
    
    Future.delayed(const Duration(seconds: 3), () {
      if (mounted) {
        setState(() => _isReceiving = false);
      }
    });
  }
  
  void _startTransmission() {
    if (!_isConnected || _isTalking) return;
    
    setState(() {
      _isTalking = true;
      _statusMessage = "TRANSMITTING...";
    });
    
    // Timer para simular transmissão
    _talkTimer = Timer(const Duration(seconds: 30), () {
      _stopTransmission(); // Auto-stop após 30 segundos
    });
    
    // Adicionar à lista de mensagens
    _messages.insert(0, {
      'sender': 'YOU',
      'message': 'Transmission in progress...',
      'time': '${DateTime.now().hour.toString().padLeft(2, '0')}:${DateTime.now().minute.toString().padLeft(2, '0')}'
    });
  }
  
  void _stopTransmission() {
    if (!_isTalking) return;
    
    _talkTimer?.cancel();
    
    setState(() {
      _isTalking = false;
      _statusMessage = "Transmission sent";
      
      // Atualizar última mensagem
      if (_messages.isNotEmpty && _messages[0]['sender'] == 'YOU') {
        _messages[0]['message'] = 'Transmission complete';
      }
    });
    
    // Retornar ao estado normal após 2 segundos
    Future.delayed(const Duration(seconds: 2), () {
      if (mounted && !_isTalking) {
        setState(() => _statusMessage = "Ready to transmit");
      }
    });
  }
  
  void _changeChannel(bool next) {
    final channels = ['CH-01', 'CH-02', 'CH-03', 'CH-04', 'CH-05', 'EMG'];
    int currentIndex = channels.indexOf(_channel);
    
    setState(() {
      if (next) {
        _channel = channels[(currentIndex + 1) % channels.length];
      } else {
        _channel = channels[(currentIndex - 1 + channels.length) % channels.length];
      }
      _statusMessage = "Switched to $_channel";
    });
  }
  
  void _toggleDevice(String deviceId) {
    setState(() {
      for (var device in _devices) {
        if (device['id'] == deviceId) {
          device['active'] = !device['active'];
          break;
        }
      }
    });
  }
  
  @override
  void dispose() {
    _talkTimer?.cancel();
    _receiveTimer?.cancel();
    _waveController?.dispose();
    super.dispose();
  }
  
  // =============== COMPONENTES HTML-LIKE ===============
  
  Widget _htmlCard({required Widget child, double elevation = 2}) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(8),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.1),
            blurRadius: elevation * 2,
            spreadRadius: elevation / 2,
          ),
        ],
        border: Border.all(color: _cssColors['gray']!.withOpacity(0.2)),
      ),
      child: child,
    );
  }
  
  Widget _htmlButton({
    required String text,
    required VoidCallback onPressed,
    Color? color,
    bool fullWidth = false,
    bool disabled = false,
  }) {
    return GestureDetector(
      onTap: disabled ? null : onPressed,
      child: Container(
        width: fullWidth ? double.infinity : null,
        padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
        decoration: BoxDecoration(
          color: disabled ? _cssColors['gray'] : (color ?? _cssColors['primary']),
          borderRadius: BorderRadius.circular(6),
          boxShadow: disabled ? null : [
            BoxShadow(
              color: (color ?? _cssColors['primary']!).withOpacity(0.3),
              blurRadius: 4,
              offset: const Offset(0, 2),
            ),
          ],
        ),
        child: Center(
          child: Text(
            text,
            style: TextStyle(
              color: Colors.white,
              fontSize: 14,
              fontWeight: FontWeight.w600,
              letterSpacing: 0.5,
            ),
          ),
        ),
      ),
    );
  }
  
  Widget _htmlSlider({
    required double value,
    required ValueChanged<double> onChanged,
    String label = '',
    Color? color,
  }) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        if (label.isNotEmpty) ...[
          Text(
            label,
            style: TextStyle(
              color: _cssColors['dark'],
              fontSize: 12,
              fontWeight: FontWeight.w500,
            ),
          ),
          const SizedBox(height: 4),
        ],
        SliderTheme(
          data: SliderThemeData(
            trackHeight: 6,
            thumbShape: RoundSliderThumbShape(
              enabledThumbRadius: 10,
              disabledThumbRadius: 8,
            ),
            overlayShape: RoundSliderOverlayShape(overlayRadius: 15),
            activeTrackColor: color ?? _cssColors['primary'],
            inactiveTrackColor: _cssColors['gray']!.withOpacity(0.3),
          ),
          child: Slider(
            value: value,
            min: 0,
            max: 1,
            divisions: 10,
            label: '${(value * 100).toInt()}%',
            onChanged: onChanged,
          ),
        ),
      ],
    );
  }
  
  Widget _htmlProgressBar(double value, Color color) {
    return Container(
      height: 8,
      decoration: BoxDecoration(
        color: _cssColors['gray']!.withOpacity(0.2),
        borderRadius: BorderRadius.circular(4),
      ),
      child: FractionallySizedBox(
        widthFactor: value,
        alignment: Alignment.centerLeft,
        child: Container(
          decoration: BoxDecoration(
            color: color,
            borderRadius: BorderRadius.circular(4),
            gradient: LinearGradient(
              colors: [color, color.withOpacity(0.8)],
            ),
          ),
        ),
      ),
    );
  }
  
  // =============== LAYOUT PRINCIPAL ===============
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: _cssColors['light'],
      body: SafeArea(
        child: Column(
          children: [
            // HEADER (como navbar HTML)
            _buildHeader(),
            
            // CONTEÚDO PRINCIPAL
            Expanded(
              child: SingleChildScrollView(
                padding: const EdgeInsets.all(16),
                child: Column(
                  children: [
                    // STATUS E CONTROLES PRINCIPAIS
                    _buildStatusPanel(),
                    const SizedBox(height: 20),
                    
                    // CONTROLE DE TRANSMISSÃO
                    _buildTransmissionControl(),
                    const SizedBox(height: 20),
                    
                    // CONFIGURAÇÕES
                    _buildSettingsPanel(),
                    const SizedBox(height: 20),
                    
                    // DISPOSITIVOS CONECTADOS
                    _buildDevicesPanel(),
                    const SizedBox(height: 20),
                    
                    // HISTÓRICO DE MENSAGENS
                    _buildMessagesPanel(),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildHeader() {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
      decoration: BoxDecoration(
        color: Colors.white,
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.05),
            blurRadius: 8,
            spreadRadius: 2,
          ),
        ],
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          // Logo/Title
          Row(
            children: [
              Icon(Icons.radio, color: _cssColors['primary'], size: 24),
              const SizedBox(width: 12),
              Text(
                'HTML WALKIE-TALKIE',
                style: TextStyle(
                  color: _cssColors['dark'],
                  fontSize: 18,
                  fontWeight: FontWeight.w700,
                  letterSpacing: 1,
                ),
              ),
            ],
          ),
          
          // Status indicators
          Row(
            children: [
              // Battery
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
                decoration: BoxDecoration(
                  color: _cssColors['light'],
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Row(
                  children: [
                    Icon(
                      _batteryLevel > 0.2 ? Icons.battery_full : Icons.battery_alert,
                      color: _batteryLevel > 0.2 ? _cssColors['success'] : _cssColors['danger'],
                      size: 16,
                    ),
                    const SizedBox(width: 6),
                    Text(
                      '${(_batteryLevel * 100).toInt()}%',
                      style: TextStyle(
                        fontSize: 12,
                        fontWeight: FontWeight.w600,
                        color: _cssColors['dark'],
                      ),
                    ),
                  ],
                ),
              ),
              
              const SizedBox(width: 12),
              
              // Signal
              Container(
                padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
                decoration: BoxDecoration(
                  color: _cssColors['light'],
                  borderRadius: BorderRadius.circular(12),
                ),
                child: Row(
                  children: [
                    Icon(
                      _signalStrength > 0.5 ? Icons.signal_cellular_alt : Icons.signal_cellular_alt_1_bar,
                      color: _signalStrength > 0.5 ? _cssColors['success'] : _cssColors['warning'],
                      size: 16,
                    ),
                    const SizedBox(width: 6),
                    Text(
                      '${(_signalStrength * 100).toInt()}%',
                      style: TextStyle(
                        fontSize: 12,
                        fontWeight: FontWeight.w600,
                        color: _cssColors['dark'],
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
  
  Widget _buildStatusPanel() {
    return _htmlCard(
      elevation: 3,
      child: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Status line
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text(
                  'SYSTEM STATUS',
                  style: TextStyle(
                    color: _cssColors['dark']!.withOpacity(0.7),
                    fontSize: 12,
                    fontWeight: FontWeight.w600,
                    letterSpacing: 1,
                  ),
                ),
                Container(
                  padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
                  decoration: BoxDecoration(
                    color: _isConnected ? _cssColors['success']!.withOpacity(0.1) 
                            : _cssColors['danger']!.withOpacity(0.1),
                    borderRadius: BorderRadius.circular(12),
                    border: Border.all(
                      color: _isConnected ? _cssColors['success']! : _cssColors['danger']!,
                    ),
                  ),
                  child: Text(
                    _isConnected ? 'CONNECTED' : 'OFFLINE',
                    style: TextStyle(
                      color: _isConnected ? _cssColors['success'] : _cssColors['danger'],
                      fontSize: 11,
                      fontWeight: FontWeight.w700,
                    ),
                  ),
                ),
              ],
            ),
            
            const SizedBox(height: 15),
            
            // Status message
            Text(
              _statusMessage,
              style: TextStyle(
                color: _cssColors['dark'],
                fontSize: 20,
                fontWeight: FontWeight.w700,
                height: 1.3,
              ),
            ),
            
            const SizedBox(height: 10),
            
            // Channel selector
            Row(
              children: [
                _htmlButton(
                  text: '◀',
                  onPressed: () => _changeChannel(false),
                  color: _cssColors['gray'],
                ),
                
                Expanded(
                  child: Container(
                    margin: const EdgeInsets.symmetric(horizontal: 10),
                    padding: const EdgeInsets.symmetric(vertical: 15),
                    decoration: BoxDecoration(
                      color: _cssColors['primary']!.withOpacity(0.05),
                      borderRadius: BorderRadius.circular(8),
                      border: Border.all(color: _cssColors['primary']!.withOpacity(0.2)),
                    ),
                    child: Center(
                      child: Text(
                        _channel,
                        style: TextStyle(
                          color: _cssColors['primary'],
                          fontSize: 28,
                          fontWeight: FontWeight.w800,
                          letterSpacing: 2,
                        ),
                      ),
                    ),
                  ),
                ),
                
                _htmlButton(
                  text: '▶',
                  onPressed: () => _changeChannel(true),
                  color: _cssColors['gray'],
                ),
              ],
            ),
            
            const SizedBox(height: 15),
            
            // Progress indicators
            Row(
              children: [
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Battery',
                        style: TextStyle(
                          color: _cssColors['dark']!.withOpacity(0.6),
                          fontSize: 12,
                        ),
                      ),
                      const SizedBox(height: 4),
                      _htmlProgressBar(_batteryLevel, _cssColors['success']!),
                    ],
                  ),
                ),
                
                const SizedBox(width: 20),
                
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Signal',
                        style: TextStyle(
                          color: _cssColors['dark']!.withOpacity(0.6),
                          fontSize: 12,
                        ),
                      ),
                      const SizedBox(height: 4),
                      _htmlProgressBar(_signalStrength, _cssColors['primary']!),
                    ],
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
  
  Widget _buildTransmissionControl() {
    return _htmlCard(
      child: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          children: [
            Text(
              'TRANSMISSION CONTROL',
              style: TextStyle(
                color: _cssColors['dark']!.withOpacity(0.7),
         
