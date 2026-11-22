# ======================================
# BANK AO DEMO - PHAN 1/3
# Telegram Bot - Python - Pydroid 3
# ======================================

import sqlite3
import logging
import secrets
from datetime import datetime

from telegram import (
    Update,
    InlineKeyboardButton,
    InlineKeyboardMarkup,
    ReplyKeyboardMarkup
)
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    CallbackQueryHandler,
    MessageHandler,
    ContextTypes,
    filters
)

# ===== CAU HINH =====

TOKEN = "8829368149:AAGPEpvRX4IO_QjU9cMNc4c3F59_c_WnIYg"

# Thay bang Telegram ID cua Admin
ADMIN_ID = 6491602779

DB_NAME = "vanbankbaogame_bot"

logging.basicConfig(
    format="%(asctime)s - %(levelname)s - %(message)s",
    level=logging.INFO
)

# ===== DATABASE =====

def connect_db():
    return sqlite3.connect(DB_NAME)


def init_db():
    with connect_db() as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS users (
                user_id INTEGER PRIMARY KEY,
                username TEXT,
                full_name TEXT,
                va_number TEXT UNIQUE,
                balance INTEGER DEFAULT 0,
                status TEXT DEFAULT 'pending',
                created_at TEXT
            )
        """)

        conn.execute("""
            CREATE TABLE IF NOT EXISTS transactions (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER,
                tx_type TEXT,
                amount INTEGER,
                note TEXT,
                created_at TEXT
            )
        """)

        conn.commit()


# ===== HAM TIEN ICH =====

def now():
    return datetime.now().strftime(
        "%d/%m/%Y %H:%M:%S"
    )


def money(amount):
    return f"{amount:,}".replace(",", ".") + " DEMO"


def get_user(user_id):
    with connect_db() as conn:
        conn.row_factory = sqlite3.Row
        row = conn.execute(
            "SELECT * FROM users WHERE user_id=?",
            (user_id,)
        ).fetchone()
        return dict(row) if row else None


def create_va():
    # Day la ma VA DEMO, khong phai STK that.
    while True:
        number = "DE" + str(
            secrets.randbelow(9000000000) + 1000000000
        )

        with connect_db() as conn:
            exists = conn.execute(
                "SELECT 1 FROM users WHERE va_number=?",
                (number,)
            ).fetchone()

        if not exists:
            return number


def register_user(tg_user):
    user_id = tg_user.id

    existing = get_user(user_id)
    if existing:
        return existing

    username = tg_user.username or ""
    full_name = tg_user.full_name or ""
    va_number = create_va()

    with connect_db() as conn:
        conn.execute("""
            INSERT INTO users (
                user_id, username, full_name,
                va_number, balance, status, created_at
            )
            VALUES (?, ?, ?, ?, 0, 'pending', ?)
        """, (
            user_id,
            username,
            full_name,
            va_number,
            now()
        ))

        conn.commit()

    return get_user(user_id)


def add_transaction(
    user_id,
    tx_type,
    amount,
    note=""
):
    with connect_db() as conn:
        conn.execute("""
            INSERT INTO transactions (
                user_id, tx_type, amount,
                note, created_at
            )
            VALUES (?, ?, ?, ?, ?)
        """, (
            user_id,
            tx_type,
            amount,
            note,
            now()
        ))

        conn.commit()


def change_balance(
    user_id,
    amount,
    tx_type,
    note=""
):
    # amount duong: cong
    # amount am: tru
    with connect_db() as conn:
        conn.execute("BEGIN IMMEDIATE")

        row = conn.execute(
            "SELECT balance FROM users WHERE user_id=?",
            (user_id,)
        ).fetchone()

        if row is None:
            return False

        new_balance = row[0] + amount

        if new_balance < 0:
            return False

        conn.execute(
            "UPDATE users SET balance=? WHERE user_id=?",
            (new_balance, user_id)
        )

        conn.execute("""
            INSERT INTO transactions (
                user_id, tx_type, amount,
                note, created_at
            )
            VALUES (?, ?, ?, ?, ?)
        """, (
            user_id,
            tx_type,
            amount,
            note,
            now()
        ))

        conn.commit()

    return True


def is_admin(user_id):
    return user_id == ADMIN_ID


def is_approved(user_id):
    user = get_user(user_id)
    return bool(
        user and user["status"] == "approved"
    )


# ===== MENU CHINH =====

def main_menu():
    return ReplyKeyboardMarkup(
        [
            ["🏦 Tài khoản VA", "💰 Số dư"],
            ["💵 Nạp ", "💸 Rút "],
            ["📜 Lịch sử", "👤 Hồ sơ"],
            ["📞 Hỗ trợ"]
        ],
        resize_keyboard=True
    )


# ===== KET THUC PHAN 1 =====
# Dan PHAN 2 ngay ben duoi dong nay.

# ======================================
# BANK AO DEMO - PHAN 2/3
# KHACH HANG VA GIAO DICH DEMO
# ======================================

# ===== DIEU KHOAN =====

TERMS = """
🏦 BANK AO 

📜 DIEU KHOAN SU DUNG

1. Bot BIDV 10 SỐ , msb, bidv.

2. VA duoc cap , khong giao dich khac ngoai game.


3. So du .

4. Khong su dung bot muc dich khac ngoai game

5. Admin co quyen duyet, khoa va
quan ly
6.ADMIN CẤP QUYEN #@huizniecry

Nhan "Toi dong y" de gui yeu cau
cap quyen su dung.
"""


def approval_keyboard():
    return InlineKeyboardMarkup([
        [
            InlineKeyboardButton(
                "✅ Tôi đồng ý",
                callback_data="agree_terms"
            )
        ]
    ])


# ===== START =====

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    tg_user = update.effective_user
    user = register_user(tg_user)

    if user["status"] == "pending":
        await update.message.reply_text(
            TERMS,
            reply_markup=approval_keyboard()
        )
        return

    if user["status"] == "blocked":
        await update.message.reply_text(
            "⛔ Tai khoan cua ban da bi khoa."
        )
        return

    await update.message.reply_text(
        "🏦 BANK AO \n\n"
        "✅ Chao mung ban quay tro lai!\n"
        "Chon chuc nang ben duoi.",
        reply_markup=main_menu()
    )


# ===== DONG Y DIEU KHOAN =====

async def agree_terms(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    user_id = query.from_user.id
    user = get_user(user_id)

    if not user:
        await query.message.reply_text(
            "Vui long nhan /start truoc."
        )
        return

    if user["status"] == "approved":
        await query.message.reply_text(
            "✅ Ban da duoc cap quyen.",
            reply_markup=main_menu()
        )
        return

    if user["status"] == "blocked":
        await query.message.reply_text(
            "⛔ Tai khoan da bi khoa."
        )
        return

    await query.message.reply_text(
        "✅ Da ghi nhan dong y dieu khoan.\n"
        "⏳ Yeu cau dang cho Admin duyet."
    )

    keyboard = InlineKeyboardMarkup([
        [
            InlineKeyboardButton(
                "✅ Duyệt",
                callback_data=f"approve_{user_id}"
            ),
            InlineKeyboardButton(
                "❌ Từ chối",
                callback_data=f"reject_{user_id}"
            )
        ]
    ])

    await context.bot.send_message(
        chat_id=ADMIN_ID,
        text=(
            "🔔 YEU CAU CAP QUYEN \n\n"
            f"👤 Ten: {user['full_name']}\n"
            f"🆔 ID: {user_id}\n"
            f"🔗 Username: @{user['username']}\n"
            f"🏦 VA: {user['va_number']}\n\n"
            "Vui long kiem tra va duyet."
        ),
        reply_markup=keyboard
    )


# ===== KIEM TRA QUYEN =====

async def check_access(update: Update):
    user_id = update.effective_user.id
    user = get_user(user_id)

    if not user:
        await update.message.reply_text(
            "Vui long nhan /start."
        )
        return False

    if user["status"] != "approved":
        await update.message.reply_text(
            "⏳ Ban chua duoc cap quyen.\n"
            "Nhan /start de kiem tra."
        )
        return False

    return True


# ===== TAI KHOAN VA =====

async def show_va(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await check_access(update):
        return

    user = get_user(update.effective_user.id)

    await update.message.reply_text(
        "🏦 TAI KHOAN \n\n"
        "⚠️ HE THONG BANK AO\n\n"
        f"🔢 Ma VA: {user['va_number']}\n"
        f"👤 Chu tai khoan: {user['full_name']}\n"
        "🏛 He thong: BANK AO \n"
        f"💰 So du: {money(user['balance'])}\n\n"
        "Ma nay chi su dung trong bot."
    )


# ===== SO DU =====

async def show_balance(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await check_access(update):
        return

    user = get_user(update.effective_user.id)

    await update.message.reply_text(
        "💰 VI \n\n"
        f"👤 {user['full_name']}\n"
        f"🏦 VA: {user['va_number']}\n"
        f"💵 So du: {money(user['balance'])}\n\n"
        "⚠️ So du "
    )


# ===== HO SO =====

async def show_profile(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await check_access(update):
        return

    user = get_user(update.effective_user.id)

    await update.message.reply_text(
        "👤 HO SO KHACH HANG \n\n"
        f"🆔 ID: {user['user_id']}\n"
        f"👤 Ten: {user['full_name']}\n"
        f"🔗 Username: @{user['username']}\n"
        f"🏦 VA: {user['va_number']}\n"
        f"💰 So du: {money(user['balance'])}\n"
        f"📅 Ngay tao: {user['created_at']}\n"
        f"🔐 Trang thai: {user['status']}"
    )


# ===== NAP DEMO =====

async def deposit_demo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await check_access(update):
        return

    keyboard = InlineKeyboardMarkup([
        [
            InlineKeyboardButton(
                "10.000 ",
                callback_data="deposit_10000"
            ),
            InlineKeyboardButton(
                "50.000 ",
                callback_data="deposit_50000"
            )
        ],
        [
            InlineKeyboardButton(
                "100.000 ",
                callback_data="deposit_100000"
            ),
            InlineKeyboardButton(
                "200.000 ",
                callback_data="deposit_200000"
            )
        ]
    ])

    await update.message.reply_text(
        "💵 NAP TIEN \n\n"
        "Chon so  muon nap.\n"
        "Chuyen khoan chup bill.",
        reply_markup=keyboard
    )


async def handle_deposit(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    user_id = query.from_user.id

    if not is_approved(user_id):
        await query.message.reply_text(
            "⛔ Ban chua duoc cap quyen."
        )
        return

    try:
        amount = int(query.data.split("_")[1])
    except (ValueError, IndexError):
        return

    if amount not in (10000, 50000, 100000, 200000):
        return

    success = change_balance(
        user_id,
        amount,
        "NAP_DEMO",
        "Nap tien mo phong"
    )

    if success:
        user = get_user(user_id)

        await query.message.reply_text(
            "✅ NAP DEMO THANH CONG\n\n"
            f"💵 So nap: {money(amount)}\n"
            f"💰 So du moi: {money(user['balance'])}\n\n"
            "⚠️ Khong co tien that duoc chuyen."
        )

        if user_id != ADMIN_ID:
            await context.bot.send_message(
                ADMIN_ID,
                "🔔 GIAO DICH DEMO\n"
                f"ID: {user_id}\n"
                f"Nap: {money(amount)}"
            )


# ===== RUT DEMO =====

async def withdraw_demo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await check_access(update):
        return

    user = get_user(update.effective_user.id)

    keyboard = InlineKeyboardMarkup([
        [
            InlineKeyboardButton(
                "10.000 DEMO",
                callback_data="withdraw_10000"
            ),
            InlineKeyboardButton(
                "50.000 DEMO",
                callback_data="withdraw_50000"
            )
        ],
        [
            InlineKeyboardButton(
                "100.000 DEMO",
                callback_data="withdraw_100000"
            )
        ]
    ])

    await update.message.reply_text(
        "💸 RUT TIEN DEMO\n\n"
        f"So du: {money(user['balance'])}\n\n"
        "Chon so DEMO muon tru khoi vi.\n"
        "Khong chuyen tien ra ngan hang.",
        reply_markup=keyboard
    )


async def handle_withdraw(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    user_id = query.from_user.id

    if not is_approved(user_id):
        await query.message.reply_text(
            "⛔ Ban chua duoc cap quyen."
        )
        return

    try:
        amount = int(query.data.split("_")[1])
    except (ValueError, IndexError):
        return

    if amount not in (10000, 50000, 100000):
        return

    success = change_balance(
        user_id,
        -amount,
        "RUT_DEMO",
        "Rut tien mo phong"
    )

    if not success:
        await query.message.reply_text(
            "❌ So du DEMO khong du."
        )
        return

    user = get_user(user_id)

    await query.message.reply_text(
        "✅ RUT DEMO THANH CONG\n\n"
        f"💸 Da tru: {money(amount)}\n"
        f"💰 Con lai: {money(user['balance'])}\n\n"
        "⚠️ Khong co giao dich ngan hang that."
    )

    if user_id != ADMIN_ID:
        await context.bot.send_message(
            ADMIN_ID,
            "🔔 RUT TIEN DEMO\n"
            f"ID: {user_id}\n"
            f"So luong: {money(amount)}"
        )


# ===== LICH SU =====

async def show_history(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await check_access(update):
        return

    user_id = update.effective_user.id

    with connect_db() as conn:
        rows = conn.execute("""
            SELECT id, tx_type, amount, note, created_at
            FROM transactions
            WHERE user_id=?
            ORDER BY id DESC
            LIMIT 10
        """, (user_id,)).fetchall()

    if not rows:
        await update.message.reply_text(
            "📜 Chua co giao dich DEMO."
        )
        return

    msg = "📜 LICH SU GIAO DICH DEMO\n\n"

    for tx in rows:
        tx_id, tx_type, amount, note, date = tx

        msg += (
            f"🧾 Ma: {tx_id}\n"
            f"📌 Loai: {tx_type}\n"
            f"💵 So luong: {money(amount)}\n"
            f"📝 {note}\n"
            f"🕒 {date}\n\n"
        )

    await update.message.reply_text(msg)


# ===== HO TRO =====

async def support(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "📞 HO TRO BANK AO DEMO\n\n"
        "Lien he Admin:\n"
        f"Telegram ID: {ADMIN_ID}\n\n"
        "⚠️ Bot chi mo phong giao dich."
    )


# ===== XU LY MENU =====

async def menu_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text

    if text == "🏦 Tài khoản VA":
        await show_va(update, context)

    elif text == "💰 Số dư":
        await show_balance(update, context)

    elif text == "👤 Hồ sơ":
        await show_profile(update, context)

    elif text == "💵 Nạp DEMO":
        await deposit_demo(update, context)

    elif text == "💸 Rút DEMO":
        await withdraw_demo(update, context)

    elif text == "📜 Lịch sử":
        await show_history(update, context)

    elif text == "📞 Hỗ trợ":
        await support(update, context)


# ===== KET THUC PHAN 2 =====
# Dan PHAN 3 ngay ben duoi.


# ======================================
# BANK AO DEMO - PHAN 3/3
# ADMIN + KHOI DONG BOT
# ======================================

# ===== KIEM TRA ADMIN =====

async def require_admin(update):
    if not is_admin(update.effective_user.id):
        if update.message:
            await update.message.reply_text(
                "⛔ Ban khong co quyen Admin."
            )
        return False
    return True


# ===== MENU ADMIN =====

async def admin_menu(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not await require_admin(update):
        return

    keyboard = InlineKeyboardMarkup([
        [
            InlineKeyboardButton(
                "👥 Thống kê",
                callback_data="admin_stats"
            ),
            InlineKeyboardButton(
                "⏳ Chờ duyệt",
                callback_data="admin_pending"
            )
        ],
        [
            InlineKeyboardButton(
                "📜 Giao dịch",
                callback_data="admin_transactions"
            )
        ]
    ])

    await update.message.reply_text(
        "👑 QUAN TRI BANK AO DEMO\n\n"
        "Lenh quan ly:\n\n"
        "/duyet USER_ID\n"
        "/tuchoi USER_ID\n"
        "/congtien USER_ID SO_TIEN\n"
        "/trutien USER_ID SO_TIEN\n"
        "/khoa USER_ID\n"
        "/mokhoa USER_ID\n"
        "/xem USER_ID\n\n"
        "Tat ca so du la DEMO.",
        reply_markup=keyboard
    )


# ===== DOI TRANG THAI =====

def set_status(user_id, status):
    with connect_db() as conn:
        cursor = conn.execute(
            "UPDATE users SET status=? WHERE user_id=?",
            (status, user_id)
        )
        conn.commit()
        return cursor.rowcount > 0


async def notify_user(context, user_id, text):
    try:
        await context.bot.send_message(
            chat_id=user_id,
            text=text
        )
    except Exception as e:
        logging.warning(
            "Khong gui duoc thong bao: %s", e
        )


# ===== DUYET / TU CHOI BANG NUT =====

async def handle_approval(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE
):
    query = update.callback_query
    await query.answer()

    if not is_admin(query.from_user.id):
        await query.message.reply_text(
            "⛔ Khong co quyen."
        )
        return

    try:
        action, user_id_text = query.data.split("_", 1)
        user_id = int(user_id_text)
    except (ValueError, IndexError):
        return

    user = get_user(user_id)

    if not user:
        await query.message.reply_text(
            "❌ Khong tim thay khach."
        )
        return

    if user["status"] != "pending":
        await query.message.reply_text(
            "⚠️ Yeu cau nay da duoc xu ly."
        )
        return

    if action == "approve":
        set_status(user_id, "approved")

        await query.edit_message_text(
            "✅ DA DUYET KHACH HANG\n"
            f"ID: {user_id}"
        )

        await notify_user(
            context,
            user_id,
            "✅ Tai khoan DEMO da duoc duyet!\n"
            "Nhan /start de su dung."
        )

    elif action == "reject":
        set_status(user_id, "blocked")

        await query.edit_message_text(
            "❌ DA TU CHOI YEU CAU\n"
            f"ID: {user_id}"
        )

        await notify_user(
            context,
            user_id,
            "❌ Yeu cau su dung DEMO bi tu choi."
        )


# ===== CAC LENH ADMIN =====

async def admin_status_command(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE
):
    if not await require_admin(update):
        return

    command = update.message.text.split()[0].split("@")[0]

    if len(context.args) != 1:
        await update.message.reply_text(
            f"Cu phap: {command} USER_ID"
        )
        return

    try:
        user_id = int(context.args[0])
    except ValueError:
        await update.message.reply_text(
            "❌ USER_ID phai la so."
        )
        return

    user = get_user(user_id)

    if not user:
        await update.message.reply_text(
            "❌ Khong tim thay nguoi dung."
        )
        return

    statuses = {
        "/duyet": "approved",
        "/tuchoi": "blocked",
        "/khoa": "blocked",
        "/mokhoa": "approved"
    }

    status = statuses.get(command)

    if status is None:
        return

    set_status(user_id, status)

    await update.message.reply_text(
        "✅ Da cap nhat trang thai DEMO.\n"
        f"ID: {user_id}\n"
        f"Trang thai: {status}"
    )

    await notify_user(
        context,
        user_id,
        "🔔 Trang thai tai khoan DEMO:\n"
        f"{status}\n"
        "Nhan /start de kiem tra."
    )


# ===== CONG / TRU TIEN =====

async def admin_balance_command(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE
):
    if not await require_admin(update):
        return

    command = update.message.text.split()[0].split("@")[0]

    if len(context.args) != 2:
        await update.message.reply_text(
            f"Cu phap: {command} USER_ID SO_TIEN"
        )
        return

    try:
        user_id = int(context.args[0])
        amount = int(context.args[1])
    except ValueError:
        await update.message.reply_text(
            "❌ ID va so tien phai la so nguyen."
        )
        return

    if amount <= 0:
        await update.message.reply_text(
            "❌ So tien phai lon hon 0."
        )
        return

    if command == "/trutien":
        amount = -amount
        tx_type = "ADMIN_TRU_DEMO"
    else:
        tx_type = "ADMIN_CONG_DEMO"

    success = change_balance(
        user_id,
        amount,
        tx_type,
        "Admin dieu chinh so du thu nghiem"
    )

    if not success:
        await update.message.reply_text(
            "❌ Khong tim thay tai khoan "
            "hoac so du khong du."
        )
        return

    user = get_user(user_id)

    await update.message.reply_text(
        "✅ CAP NHAT SO DU DEMO\n\n"
        f"ID: {user_id}\n"
        f"Thay doi: {money(amount)}\n"
        f"So du moi: {money(user['balance'])}"
    )

    await notify_user(
        context,
        user_id,
        "🔔 THONG BAO VI DEMO\n\n"
        f"Thay doi: {money(amount)}\n"
        f"So du: {money(user['balance'])}\n\n"
        "⚠️ Khong phai tien that."
    )


# ===== XEM KHACH =====

async def admin_view_user(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE
):
    if not await require_admin(update):
        return

    if len(context.args) != 1:
        await update.message.reply_text(
            "Cu phap: /xem USER_ID"
        )
        return

    try:
        user_id = int(context.args[0])
    except ValueError:
        await update.message.reply_text(
            "❌ ID khong hop le."
        )
        return

    user = get_user(user_id)

    if not user:
        await update.message.reply_text(
            "❌ Khong tim thay khach."
        )
        return

    await update.message.reply_text(
        "👤 THONG TIN KHACH DEMO\n\n"
        f"ID: {user['user_id']}\n"
        f"Ten: {user['full_name']}\n"
        f"Username: @{user['username']}\n"
        f"VA: {user['va_number']}\n"
        f"So du: {money(user['balance'])}\n"
        f"Trang thai: {user['status']}\n"
        f"Ngay tao: {user['created_at']}"
    )


# ===== THONG KE =====

def get_statistics():
    with connect_db() as conn:
        total = conn.execute(
            "SELECT COUNT(*) FROM users"
        ).fetchone()[0]

        approved = conn.execute(
            "SELECT COUNT(*) FROM users "
            "WHERE status='approved'"
        ).fetchone()[0]

        pending = conn.execute(
            "SELECT COUNT(*) FROM users "
            "WHERE status='pending'"
        ).fetchone()[0]

        blocked = conn.execute(
            "SELECT COUNT(*) FROM users "
            "WHERE status='blocked'"
        ).fetchone()[0]

        transactions = conn.execute(
            "SELECT COUNT(*) FROM transactions"
        ).fetchone()[0]

    return total, approved, pending, blocked, transactions


# ===== NUT QUAN TRI =====

async def admin_callbacks(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE
):
    query = update.callback_query
    await query.answer()

    if not is_admin(query.from_user.id):
        await query.message.reply_text(
            "⛔ Khong co quyen Admin."
        )
        return

    action = query.data

    if action == "admin_stats":
        total, approved, pending, blocked, tx_count = (
            get_statistics()
        )

        await query.message.reply_text(
            "📊 THONG KE BANK AO DEMO\n\n"
            f"👥 Tong khach: {total}\n"
            f"✅ Da duyet: {approved}\n"
            f"⏳ Cho duyet: {pending}\n"
            f"🔒 Da khoa: {blocked}\n"
            f"📜 Giao dich: {tx_count}"
        )

    elif action == "admin_pending":
        with connect_db() as conn:
            rows = conn.execute(
                "SELECT user_id, full_name "
                "FROM users WHERE status='pending' "
                "LIMIT 20"
            ).fetchall()

        if not rows:
            await query.message.reply_text(
                "✅ Khong co khach cho duyet."
            )
            return

        for user_id, full_name in rows:
            keyboard = InlineKeyboardMarkup([
                [
                    InlineKeyboardButton(
                        "✅ Duyệt",
                        callback_data=f"approve_{user_id}"
                    ),
                    InlineKeyboardButton(
                        "❌ Từ chối",
                        callback_data=f"reject_{user_id}"
                    )
                ]
            ])

            await query.message.reply_text(
                f"👤 {full_name}\nID: {user_id}",
                reply_markup=keyboard
            )

    elif action == "admin_transactions":
        with connect_db() as conn:
            rows = conn.execute("""
                SELECT id, user_id, tx_type,
                       amount, created_at
                FROM transactions
                ORDER BY id DESC
                LIMIT 10
            """).fetchall()

        if not rows:
            await query.message.reply_text(
                "📜 Chua co giao dich DEMO."
            )
            return

        msg = "📜 GIAO DICH DEMO GAN NHAT\n\n"

        for tx_id, uid, tx_type, amount, date in rows:
            msg += (
                f"#{tx_id} | ID: {uid}\n"
                f"{tx_type}: {money(amount)}\n"
                f"{date}\n\n"
            )

        await query.message.reply_text(msg)


# ===== XU LY LOI =====

async def error_handler(
    update: object,
    context: ContextTypes.DEFAULT_TYPE
):
    logging.error(
        "Bot gap loi: %s",
        context.error,
        exc_info=context.error
    )


# ===== KHOI DONG BOT =====

def main():
    if (
        not TOKEN
        or TOKEN == "DAN_TOKEN_MOI_VAO_DAY"
    ):
        print("❌ Ban chua dien TOKEN bot!")
        return

    init_db()

    app = ApplicationBuilder().token(TOKEN).build()

    # Lenh khach hang
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("nap", deposit_demo))
    app.add_handler(CommandHandler("rut", withdraw_demo))
    app.add_handler(CommandHandler("sodu", show_balance))
    app.add_handler(CommandHandler("va", show_va))
    app.add_handler(CommandHandler("lichsu", show_history))

    # Lenh Admin
    app.add_handler(CommandHandler("admin", admin_menu))
    app.add_handler(CommandHandler(
        ["duyet", "tuchoi", "khoa", "mokhoa"],
        admin_status_command
    ))
    app.add_handler(CommandHandler(
        ["congtien", "trutien"],
        admin_balance_command
    ))
    app.add_handler(CommandHandler(
        "xem", admin_view_user
    ))

    # Xu ly nut
    app.add_handler(CallbackQueryHandler(
        agree_terms,
        pattern="^agree_terms$"
    ))

    app.add_handler(CallbackQueryHandler(
        handle_approval,
        pattern=r"^(approve|reject)_\d+$"
    ))

    app.add_handler(CallbackQueryHandler(
        handle_deposit,
        pattern=r"^deposit_\d+$"
    ))

    app.add_handler(CallbackQueryHandler(
        handle_withdraw,
        pattern=r"^withdraw_\d+$"
    ))

    app.add_handler(CallbackQueryHandler(
        admin_callbacks,
        pattern=r"^admin_"
    ))

    # Menu tin nhan
    app.add_handler(MessageHandler(
        filters.TEXT & ~filters.COMMAND,
        menu_handler
    ))

    app.add_error_handler(error_handler)

    print("================================")
    print("BANK AO DEMO DANG CHAY...")
    print("ADMIN ID:", ADMIN_ID)
    print("================================")

    app.run_polling(drop_pending_updates=True)


if __name__ == "__main__":
    main()
