# Admin.py
            f"<b>{html.escape(chat.title)}:</b>\n"
            f"#PINNED\n"
            f"<b>Admin:</b> {mention_html(user.id, html.escape(user.first_name))}"
        )

        return log_message


@kigcmd(command="unpin", can_disable=False)
@bot_admin
@can_pin
@user_admin(AdminPerms.CAN_PIN_MESSAGES)
@loggable
def unpin(update: Update, context: CallbackContext) -> str:
    bot = context.bot
    chat = update.effective_chat
    user = update.effective_user

    try:
        bot.unpinChatMessage(chat.id)
    except BadRequest as excp:
        if excp.message == "Chat_not_modified":
            pass
        else:
            raise

    log_message = (
        f"<b>{html.escape(chat.title)}:</b>\n"
        f"#UNPINNED\n"
        f"<b>Admin:</b> {mention_html(user.id, html.escape(user.first_name))}"
    )

    return log_message


@kigcmd(command="invitelink", can_disable=False)
@bot_admin
@user_admin(AdminPerms.CAN_INVITE_USERS)
@connection_status
def invite(update: Update, context: CallbackContext):
    bot = context.bot
    chat = update.effective_chat

    if chat.username:
        update.effective_message.reply_text(f"https://t.me/{chat.username}")
    elif chat.type in [chat.SUPERGROUP, chat.CHANNEL]:
        bot_member = chat.get_member(bot.id)
        if bot_member.can_invite_users:
            invitelink = bot.exportChatInviteLink(chat.id)
            update.effective_message.reply_text(invitelink)
        else:
            update.effective_message.reply_text(
                "I don't have access to the invite link, try changing my permissions!"
            )
    else:
        update.effective_message.reply_text(
            "I can only give you invite links for supergroups and channels, sorry!"
        )


@kigcmd(command=["admin", "admins"])
def adminlist(update: Update, _):
    administrators = update.effective_chat.get_administrators()
    text = "Admins in *{}*:".format(update.effective_chat.title or "this chat")
    for admin in administrators:
        if not admin.is_anonymous:
            user = admin.user
            name = user.mention_markdown()
            text += "\n -> {} • `{}` • `{}` • `{}`".format(name, user.id, admin.status,
                                                           escape_markdown(
                                                               admin.custom_title) if admin.custom_title else "")

    update.effective_message.reply_text(text, parse_mode=ParseMode.MARKDOWN)


def get_help(chat):
    return gs(chat, "admin_help")


__mod_name__ = "Admin"
