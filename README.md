# -*- coding: utf-8 -*-
"""
CHƯƠNG TRÌNH CHƠI BÀI TIẾN LÊN MIỀN NAM (13 LÁ)
- Nhập 13 lá bài bất kỳ từ 52 lá (bằng bàn phím hoặc bảng chọn trực quan).
- Hệ thống phân tích 39 lá còn lại và xuất ra bộ bài 13 lá THẮNG hoặc HÒA bài của bạn.
"""

import sys
import tkinter as tk
from tkinter import ttk, messagebox
import random
import re
from collections import Counter

# ================= 1. CẤU HÌNH BỘ BÀI TIẾN LÊN MIỀN NAM =================
# Thứ tự quân bài: 3 < 4 < 5 < 6 < 7 < 8 < 9 < 10 < J < Q < K < A < 2
RANKS = ['3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A', '2']
SUITS = ['♠', '♣', '♦', '♥']  # Bích < Chuồn < Rô < Cơ
SUIT_NAMES = {'♠': 'Bích', '♣': 'Chuồn', '♦': 'Rô', '♥': 'Cơ'}
SUIT_COLORS = {'♠': '#1e293b', '♣': '#047857', '♦': '#dc2626', '♥': '#e11d48'}

RANK_ORDER = {r: i + 3 for i, r in enumerate(RANKS)}  # 3=3 ... A=14, 2=15
SUIT_ORDER = {'♠': 1, '♣': 2, '♦': 3, '♥': 4}

class Card:
    def __init__(self, rank, suit):
        self.rank = rank
        self.suit = suit
        self.rank_val = RANK_ORDER[rank]
        self.suit_val = SUIT_ORDER[suit]

    def display(self):
        return f"{self.rank}{self.suit}"

    def __repr__(self):
        return self.display()

    def __eq__(self, other):
        return isinstance(other, Card) and self.rank == other.rank and self.suit == other.suit

    def __hash__(self):
        return hash((self.rank, self.suit))

    def __lt__(self, other):
        if self.rank_val != other.rank_val:
            return self.rank_val < other.rank_val
        return self.suit_val < other.suit_val


# ================= 2. PARSER NHẬP LIỆU KÝ TỰ =================
def normalize_text(text):
    text = text.lower()
    replacements = {
        'à': 'a', 'á': 'a', 'ả': 'a', 'ã': 'a', 'ạ': 'a',
        'ă': 'a', 'ằ': 'a', 'ắ': 'a', 'ẳ': 'a', 'ẵ': 'a', 'ặ': 'a',
        'â': 'a', 'ầ': 'a', 'ấ': 'a', 'ẩ': 'a', 'ẫ': 'a', 'ậ': 'a',
        'è': 'e', 'é': 'e', 'ẻ': 'e', 'ẽ': 'e', 'ẹ': 'e',
        'ê': 'e', 'ề': 'e', 'ế': 'e', 'ể': 'e', 'ễ': 'e', 'ệ': 'e',
        'ì': 'i', 'í': 'i', 'ỉ': 'i', 'ĩ': 'i', 'ị': 'i',
        'ò': 'o', 'ó': 'o', 'ỏ': 'o', 'õ': 'o', 'ọ': 'o',
        'ô': 'o', 'ồ': 'o', 'ố': 'o', 'ổ': 'o', 'ỗ': 'o', 'ộ': 'o',
        'ơ': 'o', 'ờ': 'o', 'ớ': 'o', 'ở': 'o', 'ỡ': 'o', 'ợ': 'o',
        'ù': 'u', 'ú': 'u', 'ủ': 'u', 'ũ': 'u', 'ụ': 'u',
        'ư': 'u', 'ừ': 'u', 'ứ': 'u', 'ử': 'u', 'ữ': 'u', 'ự': 'u',
        'ỳ': 'y', 'ý': 'y', 'ỷ': 'y', 'ỹ': 'y', 'ỵ': 'y',
        'đ': 'd'
    }
    for k, v in replacements.items():
        text = text.replace(k, v)
    return text

def parse_card_token(token, context_is_vn=False):
    token = token.strip()
    if not token:
        return None

    suit = None
    if '♥' in token or 'heart' in token.lower():
        suit = '♥'
    elif '♦' in token or 'diamond' in token.lower():
        suit = '♦'
    elif '♣' in token or 'club' in token.lower():
        suit = '♣'
    elif '♠' in token or 'spade' in token.lower():
        suit = '♠'

    norm = normalize_text(token).replace('♥', '').replace('♦', '').replace('♣', '').replace('♠', '')

    if not suit:
        if 'bich' in norm or norm.endswith('b'):
            suit = '♠'
            norm = norm.replace('bich', '').rstrip('b')
        elif 'ro' in norm or norm.endswith('r') or norm.endswith('d'):
            suit = '♦'
            norm = norm.replace('ro', '').rstrip('r').rstrip('d')
        elif 'chuon' in norm or 'tep' in norm or norm.endswith('t'):
            suit = '♣'
            norm = norm.replace('chuon', '').replace('tep', '').rstrip('t')
        elif 'co' in norm or norm.endswith('h'):
            suit = '♥'
            norm = norm.replace('co', '').rstrip('h')
        elif norm.endswith('s'):
            suit = '♠'
            norm = norm.rstrip('s')
        elif norm.endswith('c'):
            suit = '♥' if context_is_vn else '♣'
            norm = norm.rstrip('c')

    norm = norm.strip().upper()
    rank = None
    if norm in ['A', 'AT', 'ACE', '1']:
        rank = 'A'
    elif norm in ['J', 'BOI']:
        rank = 'J'
    elif norm in ['Q', 'DAM']:
        rank = 'Q'
    elif norm in ['K', 'GIA']:
        rank = 'K'
    elif norm in ['2', 'HEO']:
        rank = '2'
    elif norm in ['3', '4', '5', '6', '7', '8', '9', '10']:
        rank = norm

    if rank and suit:
        return Card(rank, suit)
    return None

def parse_user_input(text):
    text = text.replace(',', ' ').replace(';', ' ').replace('|', ' ')
    is_vn = any(w in normalize_text(text) for w in ['bich', 'ro', 'tep', 'chuon', 'co', 'heo', 'at'])
    tokens = [t.strip() for t in text.split() if t.strip()]
    cards = []
    i = 0
    while i < len(tokens):
        c = parse_card_token(tokens[i], context_is_vn=is_vn)
        if c:
            cards.append(c)
            i += 1
            continue
        if i + 1 < len(tokens):
            c = parse_card_token(tokens[i] + tokens[i+1], context_is_vn=is_vn)
            if c:
                cards.append(c)
                i += 2
                continue
        i += 1
    return cards


# ================= 3. LUẬT CHƠI & ĐÁNH GIÁ ĐỘ MẠNH BÀI =================
def check_toi_trang(hand):
    """Kiểm tra Tới Trắng trong Tiến Lên Miền Nam"""
    ranks = [c.rank_val for c in hand]
    rank_counts = Counter(ranks)

    # 1. Sảnh Rồng toàn phần (3 tới 2: 13 lá liên tiếp)
    unique_ranks = sorted(list(set(ranks)))
    if len(unique_ranks) == 13 and unique_ranks == list(range(3, 16)):
        return 1500000, "TỚI TRẮNG: SẢNH RỒNG TOÀN PHẦN (3 tới 2)"

    # 2. Tứ quý 2 (4 con heo)
    if rank_counts[15] == 4:
        return 1200000, "TỚI TRẮNG: TỨ QUÝ HEO (4 con 2)"

    # 3. Sảnh Rồng (3 tới Át: 12 lá liên tiếp)
    has_3_to_A = all(r in unique_ranks for r in range(3, 15))
    if has_3_to_A:
        return 1100000, "TỚI TRẮNG: SẢNH RỒNG (3 đến Át)"

    # 4. 5 đôi thông (10 lá tạo 5 đôi liên tiếp)
    pair_ranks = sorted([r for r, count in rank_counts.items() if count >= 2 and r != 15])
    max_consec = 1
    cur_consec = 1
    for i in range(1, len(pair_ranks)):
        if pair_ranks[i] == pair_ranks[i-1] + 1:
            cur_consec += 1
            max_consec = max(max_consec, cur_consec)
        else:
            cur_consec = 1
    if max_consec >= 5:
        return 1000000, "TỚI TRẮNG: 5 ĐÔI THÔNG"

    # 5. 6 đôi bất kỳ
    total_pairs = sum(count // 2 for count in rank_counts.values())
    if total_pairs >= 6:
        return 900000, "TỚI TRẮNG: 6 ĐÔI BẤT KỲ"

    # 6. Đồng màu (13 lá hoặc 12 lá cùng màu)
    red_count = sum(1 for c in hand if c.suit in ('♥', '♦'))
    if red_count >= 12:
        return 850000, f"TỚI TRẮNG: ĐỒNG HOA ĐỎ ({red_count} lá đỏ)"
    if (13 - red_count) >= 12:
        return 850000, f"TỚI TRẮNG: ĐỒNG HOA ĐEN ({13 - red_count} lá đen)"

    return 0, None

def evaluate_hand_strength(hand):
    """
    Tính điểm lực chiến bộ bài 13 lá:
    - Tới trắng: 850,000 - 1,500,000 điểm
    - Điểm thường: tính dựa trên Heo, Hàng (Tứ quý, đôi thông), Sảnh, Đôi, Sám và trừ điểm Rác.
    """
    hand = sorted(hand)
    tt_score, tt_desc = check_toi_trang(hand)
    if tt_score > 0:
        return tt_score, tt_desc, {"toi_trang": tt_desc}

    ranks = [c.rank_val for c in hand]
    rank_counts = Counter(ranks)

    score = 0
    details = {
        "heo": [],
        "hang": [],
        "sanh": [],
        "sam": [],
        "doi": [],
        "rac": []
    }

    # 1. Điểm Heo (2)
    heos = [c for c in hand if c.rank_val == 15]
    for h in heos:
        val = 110 + h.suit_val * 15 # 2♠: 125, 2♣: 140, 2♦: 155, 2♥: 170
        score += val
        details["heo"].append(str(h))

    # 2. Tìm Tứ quý
    non_2_counts = {r: c for r, c in rank_counts.items() if r != 15}
    for r, c in non_2_counts.items():
        if c == 4:
            rank_name = RANKS[r - 3]
            score += 380 + r * 10
            details["hang"].append(f"Tứ quý {rank_name}")

    # 3. Tìm 4 đôi thông & 3 đôi thông
    pair_ranks = sorted([r for r, count in non_2_counts.items() if count >= 2])
    has_4_dt = False
    for i in range(len(pair_ranks) - 3):
        if pair_ranks[i+1] == pair_ranks[i]+1 and pair_ranks[i+2] == pair_ranks[i]+2 and pair_ranks[i+3] == pair_ranks[i]+3:
            names = f"{RANKS[pair_ranks[i]-3]} tới {RANKS[pair_ranks[i+3]-3]}"
            details["hang"].append(f"4 đôi thông ({names})")
            score += 550
            has_4_dt = True
            break
    
    if not has_4_dt:
        for i in range(len(pair_ranks) - 2):
            if pair_ranks[i+1] == pair_ranks[i]+1 and pair_ranks[i+2] == pair_ranks[i]+2:
                names = f"{RANKS[pair_ranks[i]-3]} tới {RANKS[pair_ranks[i+2]-3]}"
                details["hang"].append(f"3 đôi thông ({names})")
                score += 260
                break

    # 4. Tìm Sảnh (không tính 2)
    unique_non_2 = sorted(list(set(r for r in ranks if r != 15)))
    straights = []
    temp_s = []
    for r in unique_non_2:
        if not temp_s or r == temp_s[-1] + 1:
            temp_s.append(r)
        else:
            if len(temp_s) >= 3:
                straights.append(list(temp_s))
            temp_s = [r]
    if len(temp_s) >= 3:
        straights.append(list(temp_s))

    for st in straights:
        s_len = len(st)
        score += 35 + (s_len - 3) * 40 + max(st) * 3
        st_name = f"{RANKS[st[0]-3]} - {RANKS[st[-1]-3]} ({s_len} lá)"
        details["sanh"].append(st_name)

    # 5. Sám & Đôi
    for r, count in non_2_counts.items():
        if count == 3:
            details["sam"].append(f"Sám {RANKS[r-3]}")
            score += 45 + r * 3
        elif count == 2:
            details["doi"].append(f"Đôi {RANKS[r-3]}")
            score += 15 + r * 2

    # 6. Phạt rác lẻ
    for r, count in non_2_counts.items():
        if count == 1:
            in_straight = any(r in st for st in straights)
            if not in_straight:
                details["rac"].append(RANKS[r-3])
                if r <= 7:
                    score -= 30  # rác nhỏ 3,4,5,6,7 bị trừ nặng
                elif r <= 10:
                    score -= 15  # rác trung 8,9,10
                else:
                    score += (r - 10) * 5  # J,Q,K,A giữ bài tốt

    # Xếp hạng Tier
    if score >= 550:
        tier = "Cực Kỳ Mạnh (Cấp S)"
    elif score >= 380:
        tier = "Mạnh (Cấp A)"
    elif score >= 220:
        tier = "Trung Bình (Cấp B)"
    elif score >= 120:
        tier = "Hơi Yếu (Cấp C)"
    else:
        tier = "Yếu / Nhiều Rác (Cấp D)"

    desc = f"Lực bài: {score} điểm [{tier}]"
    return score, desc, details


# ================= 4. THUẬT TOÁN TỔNG HỢP BÀI TỪ 39 LÁ CÒN LẠI =================
def synthesize_strongest_hand(remaining_39):
    """
    Tự động xây dựng bộ bài mạnh nhất có thể từ 39 lá bài còn lại
    bằng cách gom Heo, Tứ quý, Đôi thông, Sảnh dài và các đôi/sám to nhất.
    """
    selected = []
    pool = list(remaining_39)

    # 1. Lấy tất cả con Heo (2)
    heos = [c for c in pool if c.rank_val == 15]
    selected.extend(heos)
    for h in heos:
        pool.remove(h)

    # 2. Tìm Tứ quý trong pool
    pool_counts = Counter(c.rank_val for c in pool)
    for r, count in pool_counts.items():
        if count == 4 and len(selected) + 4 <= 13:
            tq = [c for c in pool if c.rank_val == r]
            selected.extend(tq)
            for c in tq:
                pool.remove(c)

    # 3. Tìm Sảnh dài nhất
    pool_unique = sorted(list(set(c.rank_val for c in pool if c.rank_val != 15)), reverse=True)
    best_straight = []
    temp = []
    for r in pool_unique:
        if not temp or r == temp[-1] - 1:
            temp.append(r)
        else:
            if len(temp) > len(best_straight):
                best_straight = list(temp)
            temp = [r]
    if len(temp) > len(best_straight):
        best_straight = list(temp)

    if len(best_straight) >= 3 and len(selected) + len(best_straight) <= 13:
        for r in best_straight:
            # Lấy 1 lá có rank r
            card = next(c for c in pool if c.rank_val == r)
            selected.append(card)
            pool.remove(card)

    # 4. Lấy các quân bài to nhất còn lại (ưu tiên Đôi/Sám to, Át, K, Q)
    pool = sorted(pool, key=lambda c: (c.rank_val, c.suit_val), reverse=True)
    needed = 13 - len(selected)
    selected.extend(pool[:needed])

    return sorted(selected)


# ================= 5. GIAO DIỆN ĐỒ HỌA (TKINTER) =================
class CardCanvas(tk.Canvas):
    """Vẽ 13 lá bài xếp chồng tự nhiên như cầm trên tay"""
    def __init__(self, parent, **kwargs):
        super().__init__(parent, bg="#1e1e2e", highlightthickness=0, height=130, **kwargs)

    def draw_hand(self, cards):
        self.delete("all")
        if not cards:
            self.create_text(250, 65, text="Chưa có bài", font=("Segoe UI", 12), fill="#6c7086")
            return

        sorted_cards = sorted(cards)
        cw, ch = 52, 90
        spacing = 30  # khoảng cách gối đầu
        start_x = 20
        start_y = 20

        for idx, card in enumerate(sorted_cards):
            x = start_x + idx * spacing
            y = start_y
            color = SUIT_COLORS.get(card.suit, '#1e293b')

            # Vẽ lá bài trắng bo góc
            self.create_rectangle(x, y, x + cw, y + ch, fill="#ffffff", outline="#cbd5e1", width=1.5)

            # Số & chất góc trên
            self.create_text(x + 5, y + 4, text=card.rank, font=("Segoe UI", 10, "bold"), fill=color, anchor="nw")
            self.create_text(x + 5, y + 18, text=card.suit, font=("Segoe UI", 9), fill=color, anchor="nw")

            # Biểu tượng lớn ở giữa
            self.create_text(x + cw/2, y + ch/2 + 4, text=card.suit, font=("Segoe UI", 20), fill=color, anchor="center")

            # Góc dưới (chỉ hiện rõ ở lá cuối cùng)
            if idx == len(sorted_cards) - 1:
                self.create_text(x + cw - 5, y + ch - 5, text=card.rank, font=("Segoe UI", 10, "bold"), fill=color, anchor="se")


class TienLenApp:
    def __init__(self, root):
        self.root = root
        self.root.title("🃏 TIẾN LÊN MIỀN NAM - TÌM BỘ BÀI 13 LÁ THẮNG HOẶC HÒA")
        self.root.geometry("1120x820")
        self.root.minsize(1050, 780)
        self.root.configure(bg="#181825")

        self.deck = [Card(r, s) for r in RANKS for s in SUITS]
        self.selected_cards = set()

        self.user_hand = []
        self.user_score = 0
        self.user_desc = ""

        self.remaining_39 = []
        self.winning_pool = []
        self.tying_pool = []
        self.best_bot_hand = []

        self.build_ui()
        self.preset_random()

    def build_ui(self):
        # ---------------- HEADER ----------------
        hdr = tk.Frame(self.root, bg="#11111b", pady=10, padx=20)
        hdr.pack(fill="x")
        tk.Label(hdr, text="🃏 TIẾN LÊN MIỀN NAM: TÌM BỘ BÀI 13 LÁ THẮNG / HÒA TỪ 39 LÁ CÒN LẠI", 
                 font=("Segoe UI", 15, "bold"), fg="#89b4fa", bg="#11111b").pack(anchor="w")
        tk.Label(hdr, text="Bạn chọn 13 lá bất kỳ -> Chương trình phân tích lực bài và rút ra bộ bài 13 lá Thắng hoặc Hòa từ 39 lá còn lại",
                 font=("Segoe UI", 9), fg="#a6adc8", bg="#11111b").pack(anchor="w")

        main_box = tk.Frame(self.root, bg="#181825", padx=15, pady=8)
        main_box.pack(fill="both", expand=True)

        # ---------------- KHU VỰC NHẬP LIỆU ----------------
        in_box = tk.LabelFrame(main_box, text=" 📥 BƯỚC 1: CHỌN 13 LÁ BÀI CỦA BẠN ", 
                               font=("Segoe UI", 10, "bold"), fg="#89b4fa", bg="#1e1e2e", padx=12, pady=8)
        in_box.pack(fill="x", pady=(0, 8))

        # Hàng nhập text & nút bấm
        r1 = tk.Frame(in_box, bg="#1e1e2e")
        r1.pack(fill="x", pady=2)

        tk.Label(r1, text="Nhập ký tự:", font=("Segoe UI", 9, "bold"), fg="#cdd6f4", bg="#1e1e2e").pack(side="left", padx=(0, 6))
        self.txt_entry = tk.Entry(r1, font=("Segoe UI", 11), bg="#313244", fg="#ffffff", insertbackground="#ffffff")
        self.txt_entry.pack(side="left", fill="x", expand=True, padx=(0, 8))
        self.txt_entry.bind("<Return>", lambda e: self.on_analyze_from_text())

        tk.Button(r1, text="⚡ Phân Tích Bài", font=("Segoe UI", 9, "bold"), bg="#89b4fa", fg="#11111b",
                  padx=10, pady=3, relief="flat", cursor="hand2", command=self.on_analyze_from_text).pack(side="left", padx=3)

        tk.Button(r1, text="🎲 Bốc Ngẫu Nhiên", font=("Segoe UI", 9), bg="#45475a", fg="#cdd6f4",
                  padx=8, pady=3, relief="flat", cursor="hand2", command=self.preset_random).pack(side="left", padx=2)

        # Preset đặc biệt
        tk.Button(r1, text="🐉 Thử Sảnh Rồng", font=("Segoe UI", 9), bg="#585b70", fg="#f9e2af",
                  padx=8, pady=3, relief="flat", cursor="hand2", command=self.preset_sanh_rong).pack(side="left", padx=2)
        tk.Button(r1, text="💣 Thử Tứ Quý 2", font=("Segoe UI", 9), bg="#585b70", fg="#f38ba8",
                  padx=8, pady=3, relief="flat", cursor="hand2", command=self.preset_tu_quy_2).pack(side="left", padx=2)

        # Bảng chọn trực quan 52 lá (Grid Picker thu gọn)
        grid_frame = tk.Frame(in_box, bg="#1e1e2e")
        grid_frame.pack(fill="x", pady=(6, 2))

        self.grid_buttons = {}
        for row_idx, suit in enumerate(SUITS):
            rf = tk.Frame(grid_frame, bg="#1e1e2e")
            rf.pack(fill="x", pady=1)
            tk.Label(rf, text=f"{suit} {SUIT_NAMES[suit]}:", width=8, anchor="w", font=("Segoe UI", 9, "bold"),
                     fg=SUIT_COLORS[suit], bg="#1e1e2e").pack(side="left")
            for rank in RANKS:
                card = Card(rank, suit)
                btn = tk.Button(rf, text=f"{rank}{suit}", width=4, font=("Segoe UI", 8),
                                bg="#313244", fg="#cdd6f4", relief="flat", cursor="hand2",
                                command=lambda c=card: self.toggle_card(c))
                btn.pack(side="left", padx=1)
                self.grid_buttons[card] = btn

        self.lbl_selected_count = tk.Label(in_box, text="Đã chọn: 0/13 lá", font=("Segoe UI", 9, "bold"), fg="#fab387", bg="#1e1e2e")
        self.lbl_selected_count.pack(anchor="w", pady=(4, 0))

        # ---------------- KHU VỰC HIỂN THỊ SO BÀI (2 CỘT) ----------------
        compare_frame = tk.Frame(main_box, bg="#181825")
        compare_frame.pack(fill="both", expand=True, pady=4)
        compare_frame.columnconfigure(0, weight=1)
        compare_frame.columnconfigure(1, weight=1)

        # Cột trái: Bài của bạn
        box_user = tk.LabelFrame(compare_frame, text=" 👤 BỘ BÀI 13 LÁ CỦA BẠN ", 
                                 font=("Segoe UI", 11, "bold"), fg="#a6e3a1", bg="#1e1e2e", padx=10, pady=8)
        box_user.grid(row=0, column=0, sticky="nsew", padx=(0, 6))

        self.canvas_user = CardCanvas(box_user)
        self.canvas_user.pack(fill="x", pady=2)

        self.lbl_user_score = tk.Label(box_user, text="", font=("Segoe UI", 12, "bold"), fg="#a6e3a1", bg="#1e1e2e")
        self.lbl_user_score.pack(anchor="w", pady=(4, 2))

        self.txt_user_detail = tk.Text(box_user, height=7, bg="#181825", fg="#cdd6f4", font=("Consolas", 9), relief="flat")
        self.txt_user_detail.pack(fill="both", expand=True)

        # Cột phải: Bài Máy rút từ 39 lá còn lại
        box_bot = tk.LabelFrame(compare_frame, text=" 🤖 BỘ BÀI 13 LÁ TỪ 39 LÁ CÒN LẠI (THẮNG / HÒA) ", 
                                font=("Segoe UI", 11, "bold"), fg="#f9e2af", bg="#1e1e2e", padx=10, pady=8)
        box_bot.grid(row=0, column=1, sticky="nsew", padx=(6, 0))

        self.canvas_bot = CardCanvas(box_bot)
        self.canvas_bot.pack(fill="x", pady=2)

        self.lbl_bot_score = tk.Label(box_bot, text="", font=("Segoe UI", 12, "bold"), fg="#f9e2af", bg="#1e1e2e")
        self.lbl_bot_score.pack(anchor="w", pady=(4, 2))

        self.txt_bot_detail = tk.Text(box_bot, height=7, bg="#181825", fg="#cdd6f4", font=("Consolas", 9), relief="flat")
        self.txt_bot_detail.pack(fill="both", expand=True)

        # Nút điều khiển bốc bài
        action_bar = tk.Frame(box_bot, bg="#1e1e2e")
        action_bar.pack(fill="x", pady=(6, 0))

        tk.Button(action_bar, text="👑 Rút Bộ Mạnh Nhất", font=("Segoe UI", 9, "bold"), bg="#f9e2af", fg="#11111b",
                  padx=10, pady=3, relief="flat", cursor="hand2", command=self.show_strongest_bot_hand).pack(side="left", padx=3)

        tk.Button(action_bar, text="🎲 Đổi Bộ Thắng Khác", font=("Segoe UI", 9, "bold"), bg="#a6e3a1", fg="#11111b",
                  padx=10, pady=3, relief="flat", cursor="hand2", command=self.show_random_win_hand).pack(side="left", padx=3)

        tk.Button(action_bar, text="🤝 Rút Bộ Cân Kèo (Hòa)", font=("Segoe UI", 9), bg="#89dceb", fg="#11111b",
                  padx=10, pady=3, relief="flat", cursor="hand2", command=self.show_closest_tie_hand).pack(side="left", padx=3)

        # Thanh trạng thái dưới cùng
        self.lbl_status_banner = tk.Label(self.root, text="", font=("Segoe UI", 12, "bold"), bg="#11111b", fg="#ffffff", pady=6)
        self.lbl_status_banner.pack(fill="x")

    def toggle_card(self, card):
        if card in self.selected_cards:
            self.selected_cards.remove(card)
        else:
            if len(self.selected_cards) >= 13:
                messagebox.showinfo("Thông báo", "Bạn đã chọn đủ 13 lá bài. Hãy bấm 'Phân Tích Bài' hoặc bỏ chọn một lá khác.")
                return
            self.selected_cards.add(card)
        self.update_grid_ui()
        if len(self.selected_cards) == 13:
            self.analyze_cards(list(self.selected_cards))

    def update_grid_ui(self):
        count = len(self.selected_cards)
        self.lbl_selected_count.config(text=f"Đã chọn: {count}/13 lá" + (" (Đủ 13 lá)" if count == 13 else ""))
        for card, btn in self.grid_buttons.items():
            if card in self.selected_cards:
                btn.config(bg="#89b4fa", fg="#11111b")
            else:
                btn.config(bg="#313244", fg="#cdd6f4")
        # Đồng bộ text input
        sorted_cards = sorted(list(self.selected_cards))
        self.txt_entry.delete(0, tk.END)
        self.txt_entry.insert(0, " ".join(c.display() for c in sorted_cards))

    def on_analyze_from_text(self):
        text = self.txt_entry.get().strip()
        cards = parse_user_input(text)
        if len(cards) < 13:
            messagebox.showwarning("Cảnh báo", f"Bạn mới nhập {len(cards)} lá bài. Cần đủ đúng 13 lá bài để phân tích Tiến Lên!")
            return
        cards = cards[:13]
        if len(set(cards)) < 13:
            messagebox.showerror("Lỗi trùng lá", "Bạn đã nhập các lá bài trùng nhau. Vui lòng chọn 13 lá khác nhau!")
            return
        self.selected_cards = set(cards)
        self.update_grid_ui()
        self.analyze_cards(cards)

    def preset_random(self):
        chosen = random.sample(self.deck, 13)
        self.selected_cards = set(chosen)
        self.update_grid_ui()
        self.analyze_cards(chosen)

    def preset_sanh_rong(self):
        # 3 tới Át + 1 lá 2
        cards = [Card(r, '♥') if r in ['A', 'K', 'Q'] else Card(r, '♠') for r in RANKS if r != '2']
        cards.append(Card('2', '♥'))
        self.selected_cards = set(cards)
        self.update_grid_ui()
        self.analyze_cards(cards)

    def preset_tu_quy_2(self):
        # 4 con 2 + 3 đôi thông
        cards = [Card('2', s) for s in SUITS]  # 4 con 2
        cards.extend([Card('7', '♠'), Card('7', '♦'), Card('8', '♣'), Card('8', '♥'), Card('9', '♠'), Card('9', '♦')])
        cards.extend([Card('3', '♠'), Card('4', '♣'), Card('5', '♦')])
        self.selected_cards = set(cards)
        self.update_grid_ui()
        self.analyze_cards(cards)

    def analyze_cards(self, cards):
        self.user_hand = sorted(cards)
        self.user_score, self.user_desc, user_dt = evaluate_hand_strength(self.user_hand)

        self.canvas_user.draw_hand(self.user_hand)
        self.lbl_user_score.config(text=f"Bài bạn: {self.user_desc}")

        # Hiển thị chi tiết bài bạn
        self.txt_user_detail.delete("1.0", tk.END)
        self.txt_user_detail.insert(tk.END, self.format_details(user_dt))

        # 39 lá còn lại
        self.remaining_39 = [c for c in self.deck if c not in self.user_hand]

        # 1. Tìm bộ mạnh nhất từ 39 lá
        self.best_bot_hand = synthesize_strongest_hand(self.remaining_39)

        # 2. Quét ngẫu nhiên 3,000 bộ bài để tìm các bộ thắng & hòa
        self.winning_pool.clear()
        self.tying_pool.clear()

        # Luôn thêm best_bot_hand vào pool nếu thắng
        b_score, b_desc, b_dt = evaluate_hand_strength(self.best_bot_hand)
        if b_score >= self.user_score:
            self.winning_pool.append((b_score, b_desc, self.best_bot_hand, b_dt))

        for _ in range(2500):
            sample = random.sample(self.remaining_39, 13)
            s_score, s_desc, s_dt = evaluate_hand_strength(sample)
            if s_score > self.user_score:
                self.winning_pool.append((s_score, s_desc, sample, s_dt))
            elif abs(s_score - self.user_score) <= 25:
                self.tying_pool.append((s_score, s_desc, sample, s_dt))

        # Hiển thị mặc định bộ mạnh nhất hoặc bộ thắng đầu tiên
        self.show_strongest_bot_hand()

    def show_strongest_bot_hand(self):
        if not self.winning_pool and not self.tying_pool:
            b_score, b_desc, b_dt = evaluate_hand_strength(self.best_bot_hand)
            self.display_bot_hand(self.best_bot_hand, b_score, b_desc, b_dt)
            return

        # Lấy bộ có điểm cao nhất
        pool = self.winning_pool if self.winning_pool else self.tying_pool
        best = max(pool, key=lambda x: x[0])
        self.display_bot_hand(best[2], best[0], best[1], best[3])

    def show_random_win_hand(self):
        if not self.winning_pool:
            messagebox.showinfo("Thông báo", "Trong 39 lá còn lại không có bộ nào thắng được bài của bạn (Bài bạn quá mạnh)!")
            return
        chosen = random.choice(self.winning_pool)
        self.display_bot_hand(chosen[2], chosen[0], chosen[1], chosen[3])

    def show_closest_tie_hand(self):
        if not self.tying_pool:
            messagebox.showinfo("Thông báo", "Không tìm thấy bộ bài nào có điểm số sát nút (Hòa) với bài của bạn.")
            return
        chosen = min(self.tying_pool, key=lambda x: abs(x[0] - self.user_score))
        self.display_bot_hand(chosen[2], chosen[0], chosen[1], chosen[3])

    def display_bot_hand(self, hand, score, desc, dt):
        self.canvas_bot.draw_hand(hand)
        self.lbl_bot_score.config(text=f"Bài máy: {desc}")

        self.txt_bot_detail.delete("1.0", tk.END)
        self.txt_bot_detail.insert(tk.END, self.format_details(dt))

        if score > self.user_score:
            self.lbl_status_banner.config(
                text=f"🏆 BỘ BÀI MÁY THẮNG! (Lực bài máy: {score} điểm > Bạn: {self.user_score} điểm)", 
                bg="#f38ba8", fg="#11111b"
            )
        elif score == self.user_score or abs(score - self.user_score) <= 15:
            self.lbl_status_banner.config(
                text=f"🤝 BỘ BÀI CÂN KÈO / HÒA! (Lực bài tương đương: Máy {score} điểm ≈ Bạn {self.user_score} điểm)", 
                bg="#f9e2af", fg="#11111b"
            )
        else:
            self.lbl_status_banner.config(
                text=f"🌟 BÀI BẠN BẤT KHẢ CHIẾN BẠI! (Bạn: {self.user_score} điểm > Bộ mạnh nhất của máy: {score} điểm)", 
                bg="#a6e3a1", fg="#11111b"
            )

    def format_details(self, dt):
        if "toi_trang" in dt:
            return f"🌟 ĐẶC BIỆT: {dt['toi_trang']}\n(Thắng tuyệt đối tất cả các bộ bài thường)"
        lines = []
        if dt.get("heo"):
            lines.append(f"🐷 Heo (2): {', '.join(dt['heo'])}")
        if dt.get("hang"):
            lines.append(f"💣 Hàng độc: {', '.join(dt['hang'])}")
        if dt.get("sanh"):
            lines.append(f"📏 Sảnh: {', '.join(dt['sanh'])}")
        if dt.get("sam"):
            lines.append(f"👥 Sám: {', '.join(dt['sam'])}")
        if dt.get("doi"):
            lines.append(f"👫 Đôi: {', '.join(dt['doi'])}")
        if dt.get("rac"):
            lines.append(f"🗑️ Rác lẻ: {', '.join(dt['rac'])} ({len(dt['rac'])} lá rác)")
        else:
            lines.append("✨ Bài sạch: Không có rác lẻ!")
        return "\n".join(lines)


if __name__ == "__main__":
    root = tk.Tk()
    app = TienLenApp(root)
    root.mainloop()
