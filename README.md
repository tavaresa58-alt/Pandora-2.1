# ============================================================================
# PANDORA EMERGÊNCIA 2026 - Sistema Ultra-Rápido para Primeiros Socorros
# Versão: 3.0 EMERGÊNCIA
# Data: 2026
# Características: Foco em velocidade, clareza e casos mais letais
# ============================================================================

import json
import os
import sys
from datetime import datetime
from typing import Dict, List, Optional

# ============================================================================
# PROTOCOLOS DE FALLBACK MÍNIMO (garantido sempre disponível)
# ============================================================================

FALLBACK_PROTOCOLS = {
    '1': {
        'id': 'cardiac_arrest',
        'name': 'PARADA CARDÍACA / PCR',
        'priority': 'CRÍTICA',
        'steps': [
            '🚨 CHAME 192 AGORA - informe parada cardíaca',
            '⚠️ Se não responde e não respira NORMALMENTE:',
            '1. Deite a vítima em superfície FIRME',
            '2. INICIE COMPRESSÕES FORTES E RÁPIDAS',
            '3. 100-120/minuto, 5-6cm profundidade',
            '4. NÃO PARE até socorro chegar ou DEA chegar',
            '💡 Dica: cante "Stayin\' Alive" para ritmo correto'
        ],
        'warning': 'Cada minuto sem RCP reduz 10% de chance de sobrevivência'
    },
    
    '2': {
        'id': 'heart_attack',
        'name': 'INFARTO / DOR NO PEITO',
        'priority': 'CRÍTICA',
        'symptoms': [
            'Dor forte no peito (aperto, pressão, queimação)',
            'Dor que vai para braço ESQUERDO, mandíbula ou costas',
            'Falta de ar, suor frio, palidez',
            'Náusea, vômito, tontura'
        ],
        'steps': [
            '🚨 CHAME 192 AGORA - informe infarto',
            '1. Sente a vítima, NÃO deixe andar',
            '2. Se tiver ASPIRINA e NÃO for alérgico: mastigar 300mg',
            '3. Se tiver NITROGLICERINA prescrita: usar conforme médico',
            '4. Monitorar: se desmaiar, verificar respiração',
            '5. PREPARE-SE para RCP se parar de respirar'
        ]
    },
    
    '3': {
        'id': 'severe_bleeding',
        'name': 'SANGRAMENTO GRAVE / HEMORRAGIA',
        'priority': 'CRÍTICA',
        'steps': [
            '🚨 CHAME 192 AGORA - informe sangramento grave',
            '1. Use luvas ou plástico para se proteger',
            '2. APLIQUE PRESSÃO DIRETA com pano limpo sobre o ferimento',
            '3. PRESSIONE COM FORÇA - use peso do corpo se necessário',
            '4. Se sangrar através: NÃO remova, adicione mais pano por cima',
            '5. Se braço/perna: ELEVE acima do coração',
            '⚠️ SE NÃO PARAR: considere TORNIQUETE (ver instruções detalhadas)'
        ],
        'torniquete_warning': 'SÓ use se: sangramento arterial (jato), múltiplas vítimas ou ambiente perigoso. Anote HORA da aplicação. NUNCA afrouxe!'
    },
    
    '4': {
        'id': 'stroke',
        'name': 'AVC / DERRAME',
        'priority': 'CRÍTICA',
        'test_fast': [
            'F - ROSTO: peça para sorrir → um lado está caído?',
            'A - BRAÇO: peça para levantar os dois → um cai ou não sobe?',
            'S - FALA: peça para repetir uma frase → está arrastada?',
            'T - TEMPO: se QUALQUER um positivo → CHAME 192 AGORA!'
        ],
        'steps': [
            '🚨 CHAME 192 AGORA - informe AVC e hora dos sintomas',
            '1. ANOTE HORA que começou (CRÍTICO para tratamento)',
            '2. NÃO dê comida, bebida ou remédios',
            '3. Deite com cabeça ELEVADA 30 graus se consciente',
            '4. Se vomitar: coloque de LADO (se não houver trauma)',
            '5. Transporte para hospital COM UNIDADE DE AVC'
        ]
    },
    
    '5': {
        'id': 'choking_adult',
        'name': 'ENGASGO ADULTO',
        'priority': 'CRÍTICA',
        'steps': [
            '1. Pergunte: "Você está engasgado?" se balançar cabeça SIM:',
            '2. Posicione-se atrás, braços ao redor da cintura',
            '3. Punho FECHADO acima do umbigo, abaixo das costelas',
            '4. PUXAR para dentro e para cima com FORÇA (Manobra de Heimlich)',
            '5. Repetir até objeto sair ou vítima DESMAIAR',
            '⚠️ Se desmaiar: INICIAR RCP IMEDIATAMENTE'
        ]
    },
    
    '6': {
        'id': 'opioid_overdose',
        'name': 'OVERDOSE / PARADA RESPIRATÓRIA',
        'priority': 'CRÍTICA',
        'steps': [
            '🚨 CHAME 192 AGORA - informe overdose possível',
            '1. Verificar respiração: menos de 8/min ou irregular',
            '2. Se tem NALOXONA: aplicar conforme instruções',
            '3. Se NÃO respira: INICIAR RCP com ventilações',
            '4. Após naloxona: monitorar 4h (overdose pode voltar)',
            '⚠️ Ronco alto ou som de sufocamento = EMERGÊNCIA'
        ]
    },
    
    '7': {
        'id': 'more_protocols',
        'name': 'MAIS PROTOCOLOS',
        'description': 'Lista completa de protocolos disponíveis:',
        'protocols': [
            'afogamento',
            'queimadura_grave',
            'trauma_craniano',
            'crise_convulsiva',
            'reacao_alergica_grave',
            'parto_emergencial',
            'hipotermia',
            'intoxicacao'
        ]
    }
}

# ============================================================================
# PANDORA EMERGENCY 2026 - CLASSE PRINCIPAL
# ============================================================================

class PANDORAEmergency2026:
    """
    Sistema ultra-rápido para emergências reais - 2026
    """
    
    def __init__(self, protocols_file: str = "protocols_2026.json"):
        """
        Inicializa com carregamento de JSON + fallback mínimo
        """
        self.protocols_file = protocols_file
        self.protocols = {}
        self.loaded_from_json = False
        self.disclaimer = """
        ⚠️⚠️⚠️ AVISO CRÍTICO ⚠️⚠️⚠️
        
        Este sistema fornece APENAS orientações de primeiros socorros.
        NÃO substitui atendimento médico profissional.
        
        🚨 EM CASO DE EMERGÊNCIA REAL:
        1. CHAME 192 IMEDIATAMENTE
        2. SIGA as instruções do operador
        3. Use este sistema APENAS como guia adicional
        
        ⏱️ TEMPO É VIDA - Não perca tempo lendo muito
        """
        
        # Tentar carregar do JSON primeiro
        self._load_protocols_from_json()
        
        # Se não carregou, usar fallback
        if not self.protocols:
            self.protocols = FALLBACK_PROTOCOLS
            print("⚠️ Usando protocolos de emergência mínimos")
        
        print("✅ Sistema de emergência carregado")
        print(f"📋 Protocolos disponíveis: {len(self.protocols)}")
    
    def _load_protocols_from_json(self):
        """
        Tenta carregar protocolos de arquivo JSON
        """
        try:
            if os.path.exists(self.protocols_file):
                with open(self.protocols_file, 'r', encoding='utf-8') as f:
                    data = json.load(f)
                    self.protocols = data.get('emergency_protocols', {})
                    self.loaded_from_json = True
                    print(f"✅ Protocolos carregados de {self.protocols_file}")
            else:
                print(f"📄 Arquivo {self.protocols_file} não encontrado")
                print("📝 Criando arquivo de protocolos padrão...")
                self._create_default_json()
                
        except Exception as e:
            print(f"⚠️ Erro ao carregar JSON: {e}")
            print("🔄 Usando protocolos de fallback")
    
    def _create_default_json(self):
        """
        Cria arquivo JSON padrão se não existir
        """
        default_data = {
            "version": "2026.1",
            "last_updated": datetime.now().strftime("%Y-%m-%d"),
            "emergency_protocols": FALLBACK_PROTOCOLS,
            "metadata": {
                "creator": "Sistema PANDORA Emergency 2026",
                "source": "Baseado em AHA 2025, Red Cross 2026, TCCC"
            }
        }
        
        try:
            with open(self.protocols_file, 'w', encoding='utf-8') as f:
                json.dump(default_data, f, ensure_ascii=False, indent=2)
            print(f"📄 Arquivo {self.protocols_file} criado com sucesso")
            self.protocols = FALLBACK_PROTOCOLS
            self.loaded_from_json = True
        except Exception as e:
            print(f"⚠️ Erro ao criar arquivo: {e}")
    
    def show_emergency_menu(self):
        """
        Mostra menu de emergência ultra-rápido
        """
        menu = f"""
{'='*60}
🚑 PANDORA EMERGÊNCIA 2026 - MENU RÁPIDO
{'='*60}

{self.disclaimer}

📋 PROTOCOLOS MAIS URGENTES (escolha um número):

1️⃣  PARADA CARDÍACA (não responde, não respira)
2️⃣  INFARTO (dor forte no peito)
3️⃣  SANGRAMENTO GRAVE (jato ou encharca pano rápido)
4️⃣  AVC / DERRAME (face caída, braço fraco, fala arrastada)
5️⃣  ENGASGO ADULTO (não consegue respirar/falar)
6️⃣  OVERDOSE / PARADA RESPIRATÓRIA
7️⃣  MAIS PROTOCOLOS...

0️⃣  SAIR

{'='*60}
Digite o NÚMERO da emergência (1-7) ou 0 para sair:
        """
        return menu
    
    def get_protocol(self, choice: str) -> str:
        """
        Retorna protocolo formatado com disclaimer forte
        """
        if choice == '0':
            return "Saindo... Lembre-se: para emergências reais, CHAME 192!"
        
        if choice not in self.protocols:
            return f"❌ Opção {choice} inválida. Digite número de 1 a 7"
        
        protocol = self.protocols[choice]
        
        # Formatar saída
        output = f"""
{'='*60}
🚨 EMERGÊNCIA: {protocol['name']}
{'='*60}

⚠️⚠️⚠️ ATENÇÃO: LIGUE 192 AGORA MESMO ⚠️⚠️⚠️
Informe: "{protocol['name']}" e siga instruções do operador

⏱️ TEMPO CRÍTICO: Aja RAPIDAMENTE

{'='*60}
📋 O QUE FAZER (em ordem):
"""
        
        # Adicionar sintomas se existirem
        if 'symptoms' in protocol:
            output += "\n🔍 SINAIS TÍPICOS:\n"
            for symptom in protocol['symptoms']:
                output += f"• {symptom}\n"
        
        # Adicionar passos
        output += "\n🚀 AÇÕES IMEDIATAS:\n"
        for step in protocol.get('steps', []):
            output += f"{step}\n"
        
        # Teste FAST para AVC
        if choice == '4':
            output += "\n⚡ TESTE RÁPIDO AVC (FAST):\n"
            for item in protocol.get('test_fast', []):
                output += f"{item}\n"
        
        # Aviso sobre torniquete
        if choice == '3' and 'torniquete_warning' in protocol:
            output += f"\n🩹 SOBRE TORNIQUETE:\n{protocol['torniquete_warning']}\n"
        
        # Lista de protocolos adicionais
        if choice == '7':
            output += "\n📚 TODOS PROTOCOLOS DISPONÍVEIS:\n"
            for proto_id in protocol.get('protocols', []):
                output += f"• {proto_id}\n"
            output += "\nDigite o nome exato do protocolo acima: "
        
        # Adicionar disclaimer final
        output += f"""
{'='*60}
⚠️ LEMBRE-SE:
1. CHAMOU 192? Se não, LIGUE AGORA: 📞 192
2. Proteja-se primeiro (cena segura, luvas se possível)
3. Priorize: chamar ajuda → RCP/compressão → resto
4. Não mova vítima de trauma sem necessidade

🕒 Hora do protocolo: {datetime.now().strftime('%H:%M:%S')}
{'='*60}

👉 Digite 'menu' para voltar ou 'sair' para encerrar
        """
        
        return output
    
    def quick_diagnosis(self, symptom: str) -> str:
        """
        Diagnóstico ultra-rápido por palavra-chave
        """
        symptom_lower = symptom.lower()
        
        # Mapeamento rápido de sintomas para protocolos
        symptom_map = {
            'parada': '1',
            'não respira': '1',
            'cardíaca': '1',
            'infarto': '2',
            'dor peito': '2',
            'aperto peito': '2',
            'sangramento': '3',
            'sangrando': '3',
            'hemorragia': '3',
            'avc': '4',
            'derrame': '4',
            'face caída': '4',
            'engasgo': '5',
            'engasgado': '5',
            'overdose': '6',
            'naloxona': '6'
        }
        
        for key, protocol_id in symptom_map.items():
            if key in symptom_lower:
                return self.get_protocol(protocol_id)
        
        return f"""
❌ Não identifiquei emergência urgente.

Sintomas reconhecíveis rapidamente:
• "não respira" → Parada cardíaca
• "dor peito" → Infarto  
• "sangrando muito" → Hemorragia
• "face caída" → AVC

Digite UM desses sintomas ou 'menu' para ver opções completas.
        """
    
    def show_full_protocol(self, protocol_name: str) -> str:
        """
        Busca protocolo pelo nome (para opção 7)
        """
        # Buscar em todos os protocolos
        for key, proto in self.protocols.items():
            if protocol_name.lower() in proto.get('id', '').lower():
                return self.get_protocol(key)
        
        return f"❌ Protocolo '{protocol_name}' não encontrado. Digite 'menu' para lista."

# ============================================================================
# INTERFACE ULTRA-RÁPIDA
# ============================================================================

class EmergencyInterface2026:
    """
    Interface simplificada para emergências reais
    """
    
    def __init__(self):
        self.system = PANDORAEmergency2026()
        self.running = True
    
    def clear_screen(self):
        """Tenta limpar a tela (funciona em maioria dos dispositivos)"""
        os.system('cls' if os.name == 'nt' else 'clear')
    
    def show_welcome(self):
        """Tela inicial ultra-rápida"""
        self.clear_screen()
        welcome = f"""
{'='*60}
🚑🚑🚑 PANDORA EMERGÊNCIA 2026 🚑🚑🚑
Sistema Ultra-Rápido para Primeiros Socorros
{'='*60}

⚠️⚠️⚠️ EMERGÊNCIA REAL AGORA? ⚠️⚠️⚠️

Se alguém está:
• INCONSCIENTE e NÃO RESPIRA → Digite '1'
• com DOR FORTE NO PEITO → Digite '2'  
• SANGRANDO MUITO → Digite '3'
• com FACE CAÍDA / FALA ARRASTADA → Digite '4'

Ou escolha:
• 'menu' → Ver todas opções
• 'sintoma' → Descrever sintoma rápido
• 'sair' → Encerrar

📞 LEMBRE-SE: EMERGÊNCIA REAL = LIGAR 192 PRIMEIRO!
{'='*60}
        """
        print(welcome)
    
    def process_input(self, user_input: str):
        """Processa entrada do usuário de forma ultra-rápida"""
        user_input = user_input.strip()
        
        if user_input == 'sair' or user_input == '0':
            self.running = False
            return "🔄 Encerrando... CHAME 192 se precisar!"
        
        # Entrada direta para protocolos principais
        if user_input in ['1', '2', '3', '4', '5', '6']:
            return self.system.get_protocol(user_input)
        
        # Menu completo
        elif user_input == 'menu':
            return self.system.show_emergency_menu()
        
        # Diagnóstico por sintoma
        elif user_input == 'sintoma':
            return "Descreva em POUCAS PALAVRAS (ex: 'não respira', 'dor peito', 'sangrando'): "
        
        # Processar sintoma
        elif len(user_input.split()) <= 3:  # Sintoma curto
            return self.system.quick_diagnosis(user_input)
        
        # Protocolo específico (para opção 7)
        else:
            return self.system.show_full_protocol(user_input)
    
    def run(self):
        """Loop principal ultra-simplificado"""
        self.show_welcome()
        
        while self.running:
            try:
                # Prompt ultra-simples
                user_input = input("\n🚨 EMERGÊNCIA? Digite (1-7/menu/sintoma/sair): ").strip()
                
                # Processar
                response = self.process_input(user_input)
                
                # Mostrar resposta
                if response:
                    print(response)
                    
                    # Se não for pergunta, pausa breve
                    if '?' not in response:
                        print("\n" + "-"*40)
                
            except KeyboardInterrupt:
                print("\n\n⚠️ Interrompido. CHAME 192 se for emergência real!")
                self.running = False
            except Exception as e:
                print(f"\n❌ Erro: {e}")
                print("🔄 Reiniciando interface...")
                self.show_welcome()

# ============================================================================
# FUNÇÃO PRINCIPAL
# ============================================================================

def main_emergency():
    """
    Ponto de entrada para modo emergência 2026
    """
    print("🚀 Iniciando PANDORA EMERGÊNCIA 2026...")
    print("⚡ Modo ultra-rápido ativado")
    print("📞 Lembre-se: para emergências reais, LIGUE 192 PRIMEIRO!\n")
    
    # Criar e rodar interface
    interface = EmergencyInterface2026()
    interface.run()
    
    # Mensagem final
    print("\n" + "="*60)
    print("Obrigado por usar PANDORA EMERGÊNCIA 2026")
    print("Sistema desenvolvido para salvar vidas")
    print("="*60)
    print("🆘 EMERGÊNCIA? LIGUE: 192 (SAMU) | 193 (Bombeiros)")
    print("="*60)

# ============================================================================
# EXECUÇÃO DIRETA
# ============================================================================

if __name__ == "__main__":
    # Verificar se está em modo interativo
    try:
        main_emergency()
    except Exception as e:
        print(f"❌ Erro crítico: {e}")
        print("🔄 Usando fallback direto...")
        
        # Fallback mais básico ainda
        print("\n" + "="*60)
        print("🚨 EMERGÊNCIA - INSTRUÇÕES DIRETAS:")
        print("="*60)
        print("1. CHAME 192 IMEDIATAMENTE")
        print("2. INFORME:")
        print("   • Onde estão (endereço exato)")
        print("   • O que aconteceu")
        print("   • Quantas vítimas")
        print("   • Se estão conscientes/respirando")
        print("3. SIGA INSTRUÇÕES DO OPERADOR")
        print("="*60)
        print("📋 Se parou de respirar: INICIAR RCP")
        print("💓 30 compressões fortes + 2 ventilações")
        print("🔄 NÃO PARAR até socorro chegar")
        print("="*60)
