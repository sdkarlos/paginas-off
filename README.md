import React, { useState, useMemo } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, ScrollView, Image, Share, Alert, Modal } from 'react-native';
import { StatusBar } from 'expo-status-bar';
import { ChevronLeft, UserPlus, MessageSquare, Copy, Settings, Bell, BellOff, UserX, Flag, Undo2, MoreVertical, X } from 'lucide-react-native';
import { LinearGradient } from 'expo-linear-gradient';
import { assets } from '../assets';
import { CartoonButton } from '../components';
import ChatModal from '../components/ChatModal';

// --- Função Auxiliar de Cor ---
const darkenColor = (color: string, percent: number) => {
  let num = parseInt(color.replace("#", ""), 16),
    amt = Math.round(2.55 * percent),
    R = (num >> 16) - amt,
    G = ((num >> 8) & 0x00FF) - amt,
    B = (num & 0x0000FF) - amt;
  return "#" + (0x1000000 + (R<255?R<1?0:R:255)*0x10000 + (G<255?G<1?0:G:255)*0x100 + (B<255?B<1?0:B:255)).toString(16).slice(1);
};

// --- Simulação de Dados ---
const mascotImages = [assets.mascots.mascot1_1, assets.mascots.mascot2_1, assets.mascots.mascot3_1, assets.mascots.mascot4_1];

const friendsData = [
    { id: '1', name: 'Sonic', avatar: mascotImages[0], status: 'online' },
    { id: '2', name: 'Tails', avatar: mascotImages[1], status: 'away' },
    { id: '3', name: 'Knuckles', avatar: mascotImages[2], status: 'offline' },
    { id: '4', name: 'AmyRose', avatar: mascotImages[3], status: 'online' },
    { id: '5', name: 'Shadow', avatar: mascotImages[0], status: 'offline' },
    { id: '6', name: 'Rouge', avatar: mascotImages[1], status: 'away' },
    { id: '7', name: 'Silver', avatar: mascotImages[2], status: 'playing' },
    { id: '8', name: 'Blaze', avatar: mascotImages[3], status: 'playing' },
];

type FriendAction = 'attention' | 'mute' | 'block' | 'report';

interface FriendWithActions {
    id: string;
    name: string;
    avatar: string;
    status: 'online' | 'away' | 'offline' | 'playing';
    actions?: FriendAction[];
}

const inviteCode = "YUBI-A4B8C";
const inviteLink = "https://yubiapp.com/invite?code=YUBI-A4B8C";

// --- Componentes ---

const FriendItem = ({ avatar, name, status, actions = [], onPress, onActionPress, onMenuPress }: { 
    avatar: string, 
    name: string, 
    status: 'online' | 'away' | 'offline' | 'playing', 
    actions?: FriendAction[], 
    onPress: () => void,
    onActionPress: (action: FriendAction) => void,
    onMenuPress: () => void
}) => {
    const statusInfo = {
        online: { color: '#4ade80', text: 'Online' },
        away: { color: '#facc15', text: 'Ausente' },
        offline: { color: '#6b7280', text: 'Offline' },
        playing: { color: '#8b5cf6', text: 'Jogando IUBY' },
    }[status];

    const getCardColors = (status: string) => {
        switch (status) {
            case 'online':
                return ['#10b981', '#047857']; // Verde vivo
            case 'away':
                return ['#facc15', '#f97316']; // Amarelo do lootbox
            case 'playing':
                return ['#8b5cf6', '#6d28d9']; // Roxo para jogando
            default:
                return ['#4B5563', '#1F2937']; // Cinza padrão para offline
        }
    };

    const gradient = getCardColors(status) as [string, string];
    const borderGradientColors = gradient.map(c => darkenColor(c, 15)) as [string, string, ...string[]];
    const bottomBorderColor = darkenColor(gradient[1], 15);

    return (
        <TouchableOpacity onPress={onPress}>
            <LinearGradient colors={borderGradientColors} style={styles.itemSideBorder}>
                <View style={[styles.itemBottomBorder, { backgroundColor: bottomBorderColor }]}>
                    <LinearGradient colors={gradient} style={styles.itemContainer}>
                        <View>
                            <Image source={avatar} style={styles.avatar} />
                            <View style={[styles.statusIndicator, { backgroundColor: statusInfo.color }]} />
                        </View>
                        <View style={styles.friendInfo}>
                            <Text style={styles.nameText} numberOfLines={1}>{name}</Text>
                            <Text style={[styles.statusText, { color: statusInfo.color }]}>{statusInfo.text}</Text>
                        </View>
                        
                        {/* Ícones de ações aplicadas */}
                        <View style={styles.actionIconsContainer}>
                            {actions.includes('attention') ? (
                                <TouchableOpacity 
                                    style={styles.actionIcon} 
                                    onPress={() => onActionPress('attention')}
                                >
                                    <Bell size={12} color="#facc15" />
                                </TouchableOpacity>
                            ) : null}
                            {actions.includes('mute') ? (
                                <TouchableOpacity 
                                    style={styles.actionIcon} 
                                    onPress={() => onActionPress('mute')}
                                >
                                    <BellOff size={12} color="#6b7280" />
                                </TouchableOpacity>
                            ) : null}
                            {actions.includes('block') ? (
                                <TouchableOpacity 
                                    style={styles.actionIcon} 
                                    onPress={() => onActionPress('block')}
                                >
                                    <UserX size={12} color="#ef4444" />
                                </TouchableOpacity>
                            ) : null}
                            {actions.includes('report') ? (
                                <TouchableOpacity 
                                    style={styles.actionIcon} 
                                    onPress={() => onActionPress('report')}
                                >
                                    <Flag size={12} color="#f97316" />
                                </TouchableOpacity>
                            ) : null}
                        </View>
                        
                        <TouchableOpacity style={styles.menuButton} onPress={onMenuPress}>
                            <MoreVertical size={16} color="white" />
                        </TouchableOpacity>
                        
                        <TouchableOpacity style={styles.messageButton} onPress={onPress}>
                            <MessageSquare size={24} color="white" />
                        </TouchableOpacity>
                    </LinearGradient>
                </View>
            </LinearGradient>
        </TouchableOpacity>
    );
};

export const FriendsPage = ({ navigation }: { navigation: any }) => {
    const [activeTab, setActiveTab] = useState('Convites');
    const [chatVisible, setChatVisible] = useState(false);
    const [selectedFriend, setSelectedFriend] = useState<any>(null);
    const [userStatus, setUserStatus] = useState('online');
    const [friendsWithActions, setFriendsWithActions] = useState<FriendWithActions[]>(
        friendsData.map(friend => ({ ...friend, actions: [] }))
    );
    const [menuVisible, setMenuVisible] = useState(false);
    const [menuFriend, setMenuFriend] = useState<FriendWithActions | null>(null);

    const openChat = (friend: any) => {
        setSelectedFriend(friend);
        setChatVisible(true);
    };

    const closeChat = () => {
        setChatVisible(false);
        setSelectedFriend(null);
    };

    const showFriendMenu = (friend: FriendWithActions) => {
        setMenuFriend(friend);
        setMenuVisible(true);
    };

    const closeFriendMenu = () => {
        setMenuVisible(false);
        setMenuFriend(null);
    };

    const handleMenuAction = (action: FriendAction) => {
        if (menuFriend) {
            handleFriendAction(menuFriend.id, action);
        }
        closeFriendMenu();
    };

    const handleFriendAction = (friendId: string, action: FriendAction) => {
        setFriendsWithActions(prev => 
            prev.map(friend => {
                if (friend.id === friendId) {
                    const hasAction = friend.actions?.includes(action);
                    if (hasAction) {
                        // Desfazer ação
                        const actionMessages = {
                            attention: `Parou de chamar atenção para ${friend.name}`,
                            mute: `Som de ${friend.name} ativado`,
                            block: `${friend.name} foi desbloqueado`
                        };
                        Alert.alert(
                            'Ação desfeita',
                            actionMessages[action]
                        );
                        return {
                            ...friend,
                            actions: friend.actions?.filter(a => a !== action) || []
                        };
                    } else {
                        // Aplicar ação
                        const actionMessages = {
                            attention: `Chamando atenção de ${friend.name}`,
                            mute: `${friend.name} foi silenciado`,
                            block: `${friend.name} foi bloqueado`
                        };
                        Alert.alert(
                            'Ação aplicada',
                            actionMessages[action]
                        );
                        return {
                            ...friend,
                            actions: [...(friend.actions || []), action]
                        };
                    }
                }
                return friend;
            })
        );
    };

    const onShare = async () => {
        try {
            await Share.share({
                message: `Entre para o Yubi e ganhe prêmios! Use meu código de convite: ${inviteCode} ou acesse: ${inviteLink}`,
                url: inviteLink,
                title: 'Convite para o Yubi App'
            });
        } catch (error: any) {
            Alert.alert(error.message);
        }
    };

    return (
        <View style={styles.container}>
            <StatusBar style="light" />
            <View style={styles.header}>
                <TouchableOpacity style={styles.backButton} onPress={() => navigation.goBack()}>
                    <ChevronLeft size={28} color="white" />
                </TouchableOpacity>
                <Text style={styles.headerTitle}>Amigos</Text>
                <View style={{ width: 44 }} />
            </View>

            <View style={styles.tabContainer}>
                {['Convites', 'Amigos', 'Status'].map(tab => (
                    <TouchableOpacity key={tab} onPress={() => setActiveTab(tab)}>
                        <View style={[styles.tabButton, activeTab === tab && styles.activeTab]}>
                            <Text style={styles.tabText}>{tab}</Text>
                        </View>
                    </TouchableOpacity>
                ))}
            </View>

            <ScrollView contentContainerStyle={styles.scrollContent}>
                {activeTab === 'Convites' ? (
                    <View>
                        <View style={styles.inviteHeader}>
                            <UserPlus size={48} color="#a78bfa" />
                            <Text style={styles.inviteTitle}>Convide Amigos, Ganhe Prêmios!</Text>
                            <Text style={styles.inviteSubtitle}>Para cada amigo que se cadastrar com seu código, você ganha 500 XP e 500 WIBX!</Text>
                        </View>

                        <View style={styles.rewardsInfoCard}>
                            <View style={styles.rewardItem}>
                                <Text style={styles.rewardValue}>12</Text>
                                <Text style={styles.rewardLabel}>Convites Aceitos</Text>
                            </View>
                            <View style={styles.rewardItem}>
                                <Text style={styles.rewardValue}>6,000</Text>
                                <Text style={styles.rewardLabel}>XP Ganhos</Text>
                            </View>
                            <View style={styles.rewardItem}>
                                <Text style={styles.rewardValue}>6,000</Text>
                                <Text style={styles.rewardLabel}>WIBX Ganhos</Text>
                            </View>
                        </View>

                        <Text style={styles.codeTitle}>Seu Código de Convite</Text>
                        <View style={styles.codeCard}>
                            <Text style={styles.codeText}>{inviteCode}</Text>
                            <TouchableOpacity>
                                <Copy size={24} color="#a78bfa" />
                            </TouchableOpacity>
                        </View>
                        
                        <CartoonButton
                            text="CONVIDAR AMIGOS"
                            onPress={onShare}
                            colors={['#8b5cf6', '#4338ca']}
                            shadowColor={darkenColor('#4338ca', 15)}
                        />
                    </View>
                ) : null}

                {activeTab === 'Amigos' ? (
                     <View>
                        {friendsWithActions
                            .sort((a, b) => {
                                const statusOrder = { 'online': 0, 'playing': 1, 'away': 2, 'offline': 3 };
                                return statusOrder[a.status] - statusOrder[b.status];
                            })
                            .map(friend => (
                                <FriendItem 
                                    key={friend.id} 
                                    avatar={friend.avatar}
                                    name={friend.name}
                                    status={friend.status}
                                    actions={friend.actions}
                                    onPress={() => openChat(friend)}
                                    onActionPress={(action) => handleFriendAction(friend.id, action)}
                                    onMenuPress={() => showFriendMenu(friend)}
                                />
                            ))}
                    </View>
                ) : null}

                {activeTab === 'Status' ? (
                    <View>
                        <View style={styles.statusHeader}>
                            <Settings size={48} color="#a78bfa" />
                            <Text style={styles.statusTitle}>Configurações de Status</Text>
                            <Text style={styles.statusSubtitle}>Defina como você aparece para seus amigos</Text>
                        </View>

                        <View style={styles.currentStatusCard}>
                            <Text style={styles.currentStatusTitle}>Status Atual</Text>
                            <View style={styles.currentStatusDisplay}>
                                <View style={[styles.statusDot, { backgroundColor: userStatus === 'online' ? '#4ade80' : userStatus === 'away' ? '#facc15' : userStatus === 'offline' ? '#6b7280' : '#8b5cf6' }]} />
                                <Text style={styles.currentStatusText}>
                                    {userStatus === 'online' ? 'Online' : userStatus === 'away' ? 'Ausente' : userStatus === 'offline' ? 'Offline' : 'Jogando'}
                                </Text>
                            </View>
                        </View>

                        <View style={styles.statusOptionsContainer}>
                            <Text style={styles.statusOptionsTitle}>Alterar Status</Text>
                            
                            {[
                                { key: 'online', label: 'Online', color: '#4ade80', description: 'Disponível para conversar' },
                                { key: 'away', label: 'Ausente', color: '#facc15', description: 'Longe do dispositivo' },
                                { key: 'playing', label: 'Jogando', color: '#8b5cf6', description: 'Jogando IUBY' },
                                { key: 'offline', label: 'Offline', color: '#6b7280', description: 'Não perturbe' }
                            ].map(status => (
                                <TouchableOpacity 
                                    key={status.key}
                                    style={[styles.statusOption, userStatus === status.key && styles.activeStatusOption]}
                                    onPress={() => setUserStatus(status.key)}
                                >
                                    <View style={styles.statusOptionLeft}>
                                        <View style={[styles.statusOptionDot, { backgroundColor: status.color }]} />
                                        <View>
                                            <Text style={styles.statusOptionLabel}>{status.label}</Text>
                                            <Text style={styles.statusOptionDescription}>{status.description}</Text>
                                        </View>
                                    </View>
                                    {userStatus === status.key ? (
                                        <View style={styles.checkmark}>
                                            <Text style={styles.checkmarkText}>{'✓'}</Text>
                                        </View>
                                    ) : null}
                                </TouchableOpacity>
                            ))}
                        </View>
                    </View>
                ) : null}
            </ScrollView>
            
            {selectedFriend ? (
                <ChatModal
                    visible={chatVisible}
                    onClose={closeChat}
                    friendName={selectedFriend.name}
                    friendAvatar={selectedFriend.avatar}
                    friendStatus={selectedFriend.status}
                    friendId={selectedFriend.id}
                    friendActions={selectedFriend.actions || []}
                    onActionUpdate={(friendId, action, isApplied) => {
                        setFriendsWithActions(prev => 
                            prev.map(friend => {
                                if (friend.id === friendId) {
                                    return {
                                        ...friend,
                                        actions: isApplied 
                                            ? [...(friend.actions || []), action]
                                            : (friend.actions || []).filter(a => a !== action)
                                    };
                                }
                                return friend;
                            })
                        );
                    }}
                />
            ) : null}

            {/* Popup de ações do amigo */}
            <Modal
                visible={menuVisible}
                transparent={true}
                animationType="fade"
                onRequestClose={closeFriendMenu}
            >
                <View style={styles.modalOverlay}>
                    <View style={styles.menuPopup}>
                        <View style={styles.menuHeader}>
                            <Text style={styles.menuTitle}>
                                {menuFriend ? `Ações para ${menuFriend.name}` : 'Ações'}
                            </Text>
                            <TouchableOpacity onPress={closeFriendMenu} style={styles.closeButton}>
                                <X size={24} color="white" />
                            </TouchableOpacity>
                        </View>
                        
                        {menuFriend && (
                            <View style={styles.menuActions}>
                                <TouchableOpacity 
                                    style={styles.menuActionButton}
                                    onPress={() => handleMenuAction('attention')}
                                >
                                    <Bell size={20} color="#facc15" />
                                    <Text style={styles.menuActionText}>
                                        {menuFriend.actions?.includes('attention') ? 'Parar de Chamar Atenção' : 'Chamar Atenção'}
                                    </Text>
                                </TouchableOpacity>
                                
                                <TouchableOpacity 
                                    style={styles.menuActionButton}
                                    onPress={() => handleMenuAction('mute')}
                                >
                                    <BellOff size={20} color="#6b7280" />
                                    <Text style={styles.menuActionText}>
                                        {menuFriend.actions?.includes('mute') ? 'Ativar Notificação' : 'Silenciar'}
                                    </Text>
                                </TouchableOpacity>
                                
                                <TouchableOpacity 
                                    style={styles.menuActionButton}
                                    onPress={() => handleMenuAction('block')}
                                >
                                    <UserX size={20} color="#ef4444" />
                                    <Text style={styles.menuActionText}>
                                        {menuFriend.actions?.includes('block') ? 'Desbloquear' : 'Bloquear'}
                                    </Text>
                                </TouchableOpacity>
                            </View>
                        )}
                    </View>
                </View>
            </Modal>
        </View>
    );
};

const styles = StyleSheet.create({
    container: { flex: 1, backgroundColor: '#1A0033' },
    header: { flexDirection: 'row', alignItems: 'center', justifyContent: 'space-between', paddingHorizontal: 16, paddingTop: 50, paddingBottom: 15 },
    backButton: { padding: 8 },
    headerTitle: { color: 'white', fontSize: 22, fontWeight: 'bold' },
    tabContainer: { flexDirection: 'row', justifyContent: 'center', alignItems: 'center', marginBottom: 15, paddingHorizontal: 16, gap: 15 },
    tabButton: { paddingVertical: 10, paddingHorizontal: 20, borderRadius: 20, backgroundColor: 'rgba(78, 6, 212, 0.5)' },
    activeTab: { backgroundColor: '#6F48F5', shadowColor: '#6F48F5', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.3, shadowRadius: 4, elevation: 5 },
    tabText: { color: 'white', fontSize: 16, fontWeight: 'bold' },
    scrollContent: { paddingHorizontal: 16, paddingBottom: 100, gap: 15 },
    
    // Estilos Convites
    inviteHeader: { alignItems: 'center', paddingVertical: 20, },
    inviteTitle: { fontSize: 24, fontWeight: 'bold', color: 'white', marginTop: 10, marginBottom: 5 },
    inviteSubtitle: { fontSize: 14, color: '#d1c4e9', textAlign: 'center', maxWidth: '80%' },
    rewardsInfoCard: { flexDirection: 'row', justifyContent: 'space-around', backgroundColor: 'rgba(0,0,0,0.2)', borderRadius: 20, padding: 20, marginBottom: 25 },
    rewardItem: { alignItems: 'center' },
    rewardValue: { color: 'white', fontSize: 28, fontWeight: 'bold' },
    rewardLabel: { color: '#a78bfa', fontSize: 14 },
    codeTitle: { color: 'white', fontSize: 16, fontWeight: '600', textAlign: 'center', marginBottom: 10 },
    codeCard: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', backgroundColor: 'rgba(0,0,0,0.3)', borderRadius: 15, paddingVertical: 15, paddingHorizontal: 25, marginBottom: 25 },
    codeText: { color: '#e9d5ff', fontFamily: 'monospace', fontSize: 22, letterSpacing: 2 },

    // Estilos Amigos
    itemSideBorder: { borderRadius: 27, paddingLeft: 4.5, paddingBottom: 6 },
    itemBottomBorder: { borderRadius: 27 },
    itemContainer: { flexDirection: 'row', alignItems: 'center', padding: 7.5, borderRadius: 22.5, minHeight: 72 },
    avatar: { width: 49.5, height: 49.5, marginHorizontal: 10.5, resizeMode: 'contain' },
    statusIndicator: { width: 13.5, height: 13.5, borderRadius: 6.75, borderWidth: 1.5, borderColor: '#1F2937', position: 'absolute', bottom: 0, right: 10.5 },
    friendInfo: { flex: 1, justifyContent: 'center' },
    nameText: { color: 'white', fontSize: 15, fontWeight: '600' },
    statusText: { fontSize: 12, marginTop: 1.5 },
    messageButton: { padding: 10, marginRight: 4.5, backgroundColor: 'rgba(255,255,255,0.5)', borderRadius: 15, shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.25, shadowRadius: 3.84, elevation: 5 },
    actionIconsContainer: { flexDirection: 'row', alignItems: 'center', marginRight: 12, gap: 6 },
    actionIcon: { padding: 3, backgroundColor: 'rgba(0,0,0,0.3)', borderRadius: 12, width: 24, height: 24, alignItems: 'center', justifyContent: 'center' },
    menuButton: { padding: 8, marginRight: 6, backgroundColor: 'rgba(255,255,255,0.5)', borderRadius: 15, shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.25, shadowRadius: 3.84, elevation: 5 },
    // Estilos do popup de menu
    modalOverlay: { flex: 1, backgroundColor: 'rgba(0,0,0,0.5)', justifyContent: 'center', alignItems: 'center' },
    menuPopup: { backgroundColor: '#2A1A4A', borderRadius: 20, padding: 20, margin: 20, minWidth: 300 },
    menuHeader: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', marginBottom: 20 },
    menuTitle: { color: 'white', fontSize: 18, fontWeight: 'bold' },
    closeButton: { padding: 8, backgroundColor: 'rgba(255,255,255,0.5)', borderRadius: 15, shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.25, shadowRadius: 3.84, elevation: 5 },
    menuActions: { gap: 15 },
    menuActionButton: { flexDirection: 'row', alignItems: 'center', padding: 15, backgroundColor: 'rgba(255,255,255,0.1)', borderRadius: 15, gap: 15 },
    menuActionText: { color: 'white', fontSize: 16, fontWeight: '500' },

    // Estilos Status
    statusHeader: { alignItems: 'center', paddingVertical: 20 },
    statusTitle: { fontSize: 24, fontWeight: 'bold', color: 'white', marginTop: 10, marginBottom: 5 },
    statusSubtitle: { fontSize: 14, color: '#d1c4e9', textAlign: 'center', maxWidth: '80%' },
    currentStatusCard: { backgroundColor: 'rgba(0,0,0,0.2)', borderRadius: 20, padding: 20, marginBottom: 25 },
    currentStatusTitle: { color: 'white', fontSize: 18, fontWeight: 'bold', marginBottom: 15, textAlign: 'center' },
    currentStatusDisplay: { flexDirection: 'row', alignItems: 'center', justifyContent: 'center' },
    statusDot: { width: 12, height: 12, borderRadius: 6, marginRight: 10 },
    currentStatusText: { color: 'white', fontSize: 16, fontWeight: '600' },
    statusOptionsContainer: { marginBottom: 20 },
    statusOptionsTitle: { color: 'white', fontSize: 18, fontWeight: 'bold', marginBottom: 15 },
    statusOption: { flexDirection: 'row', alignItems: 'center', justifyContent: 'space-between', backgroundColor: 'rgba(0,0,0,0.2)', borderRadius: 15, padding: 15, marginBottom: 10 },
    activeStatusOption: { backgroundColor: 'rgba(111, 72, 245, 0.3)', borderWidth: 1, borderColor: '#6F48F5' },
    statusOptionLeft: { flexDirection: 'row', alignItems: 'center' },
    statusOptionDot: { width: 10, height: 10, borderRadius: 5, marginRight: 12 },
    statusOptionLabel: { color: 'white', fontSize: 16, fontWeight: '600' },
    statusOptionDescription: { color: '#d1c4e9', fontSize: 12, marginTop: 2 },
    checkmark: { backgroundColor: '#6F48F5', borderRadius: 12, width: 24, height: 24, alignItems: 'center', justifyContent: 'center' },
    checkmarkText: { color: 'white', fontSize: 16, fontWeight: 'bold' }
});

export default FriendsPage;
