# List of Events

All events mentioned in this list are a sub-type of [GenericEvent](https://github.com/DV8FromTheWorld/JDA/tree/master/src/main/java/net/dv8tion/jda/api/events/GenericEvent.java)

## JDA Events

<pre>
GenericEvent
├── DisconnectEvent
├── ExceptionEvent
├── ReadyEvent
├── ResumedEvent
├── ReconnectedEvent
├── ShutdownEvent
├── StatusChangeEvent <sup>(1)</sup>
├── HttpRequestEvent
├── RawGatewayEvent <sup>(2)</sup>
├── GatewayPingEvent
├── MessageBulkDeleteEvent <sup>(2)</sup>
├── PrivateChannelCreateEvent
├── PrivateChannelDeleteEvent
├── UnavailableGuildJoinedEvent
├── UnavailableGuildLeaveEvent
└── UpdateEvent
</pre>

## Self Events

Fires only in relation to the currently logged in account.

<pre>
GenericSelfUpdateEvent <sup>(1)</sup>
├── SelfUpdateAvatarEvent
├── SelfUpdateDiscriminatorEvent
├── SelfUpdateEmailEvent
├── SelfUpdateMFAEvent
├── SelfUpdateMobileEvent
├── SelfUpdateNameEvent
├── SelfUpdateNitroEvent
├── SelfUpdatePhoneNumberEvent
└── SelfUpdateVerifiedEvent
</pre>


## User Events

<pre>
GenericUserEvent
├── GenericUserUpdateEvent <sup>(1)</sup>
│   ├── UserUpdateOnlineStatusEvent <sup>(3)</sup>
|   ├── UserUpdateActivityOrderEvent <sup>(3)</sup>
│   ├── UserUpdateAvatarEvent
│   ├── UserUpdateDiscriminatorEvent
│   └── UserUpdateNameEvent
├── UserActivityStartEvent <sup>(3)</sup>
├── UserActivityEndEvent <sup>(3)</sup>
└── UserTypingEvent
</pre>

## Message Events

<pre>
GenericMessageEvent
├── MessageDeleteEvent
├── MessageEmbedEvent
├── MessageReceivedEvent
├── MessageUpdateEvent
└── GenericMessageReactionEvent
    ├── MessageReactionAddEvent
    └── MessageReactionRemoveEvent
</pre>

- **MessageBulkDeleteEvent** is a special event and does not extend `GenericMessageEvent`!

### Private Message Events

<pre>
GenericPrivateMessageEvent
├── PrivateMessageDeleteEvent
├── PrivateMessageEmbedEvent
├── PrivateMessageReceivedEvent
├── PrivateMessageUpdateEvent
└── GenericPrivateMessageReactionEvent
    ├── PrivateMessageReactionAddEvent
    └── PrivateMessageReactionRemoveEvent
</pre>

## Guild Events

<pre>
GenericGuildEvent
├── GuildReadyEvent
├── GuildAvailableEvent
├── GuildUnavailableEvent
├── GuildJoinEvent
├── GuildLeaveEvent
├── GuildBanEvent
├── GuildUnbanEvent
├── GenericGuildInviteEvent
│   ├── GuildInviteCreateEvent
│   └── GuildInviteDeleteEvent
├── GenericGuildMessageEvent
│   ├── GuildMessageDeleteEvent
│   ├── GuildMessageEmbedEvent
│   ├── GuildMessageReceivedEvent
│   ├── GuildMessageUpdateEvent
│   ├── GuildMessageReactionRemoveAllEvent
│   ├── GuildMessageReactionRemoveEmoteEvent
│   └── GenericGuildMessageReactionEvent
│       ├── GuildMessageReactionAddEvent
│       └── GuildMessageReactionRemoveEvent
├── GenericGuildUpdateEvent <sup>(1)</sup>
│   ├── GuildUpdateAfkChannelEvent
│   ├── GuildUpdateAfkTimeoutEvent
│   ├── GuildUpdateExplicitContentLevelEvent
│   ├── GuildUpdateFeaturesEvent
│   ├── GuildUpdateIconEvent
│   ├── GuildUpdateMFALevelEvent
│   ├── GuildUpdateNameEvent
│   ├── GuildUpdateNotificationLevelEvent
│   ├── GuildUpdateOwnerEvent
│   ├── GuildUpdateRegionEvent
│   ├── GuildUpdateSplashEvent
│   ├── GuildUpdateSystemChannelEvent
│   ├── GuildUpdateBannerEvent
│   ├── GuildUpdateBoostCountEvent
│   ├── GuildUpdateBoostTierEvent
│   ├── GuildUpdateDescriptionEvent
│   ├── GuildUpdateMaxMembersEvent
│   ├── GuildUpdateMaxPresencesEvent
│   ├── GuildUpdateVanityCodeEvent
│   └── GuildUpdateVerificationLevelEvent
├── GenericGuildMemberEvent
│   ├── GuildMemberJoinEvent
│   ├── GuildMemberRemoveEvent
│   ├── GuildMemberRoleAddEvent
│   ├── GuildMemberRoleRemoveEvent
│   ├── GuildMemberUpdateEvent
│   └── GenericGuildMemberUpdateEvent <sup>(1)</sup>
│       ├── GuildMemberUpdateNicknameEvent
│       ├── GuildMemberUpdateAvatarEvent
│       ├── GuildMemberUpdatePendingEvent
|       ├── GuildMemberUpdateTimeOutEvent
│       └── GuildMemberUpdateBoostTimeEvent
└── GenericGuildVoiceEvent
    ├── GuildVoiceDeafenEvent
    ├── GuildVoiceGuildDeafenEvent
    ├── GuildVoiceGuildMuteEvent
    ├── GuildVoiceMuteEvent
    ├── GuildVoiceSelfDeafenEvent
    ├── GuildVoiceSelfMuteEvent
    ├── GuildVoiceStreamEvent
    ├── GuildVoiceSuppressEvent
    └── GuildVoiceUpdateEvent <sup>(1)</sup>
        ├── GuildVoiceJoinEvent
        ├── GuildVoiceLeaveEvent
        └── GuildVoiceMoveEvent
</pre>

- **UnavailableGuildJoinedEvent** and **UnavailableGuildLeaveEvent** are special events and do not extend `GenericGuildEvent`!

## Channel Events

### Text Channel Events

<pre>
GenericTextChannelEvent
├── TextChannelCreateEvent
├── TextChannelDeleteEvent
└── GenericTextChannelUpdateEvent <sup>(1)</sup>
    ├── TextChannelUpdateNameEvent
    ├── TextChannelUpdateNSFWEvent
    ├── TextChannelUpdateParentEvent
    ├── TextChannelUpdatePositionEvent
    ├── TextChannelUpdateSlowmodeEvent
    └── TextChannelUpdateTopicEvent
</pre>

### Voice Channel Events

<pre>
GenericVoiceChannelEvent
├── VoiceChannelCreateEvent
├── VoiceChannelDeleteEvent
└── GenericVoiceChannelUpdateEvent <sup>(1)</sup>
    ├── VoiceChannelUpdateBitrateEvent
    ├── VoiceChannelUpdateNameEvent
    ├── VoiceChannelUpdateParentEvent
    ├── VoiceChannelUpdatePositionEvent
    └── VoiceChannelUpdateUserLimitEvent
</pre>

### Category Events

<pre>
GenericCategoryEvent
├── CategoryCreateEvent
├── CategoryDeleteEvent
└── GenericCategoryUpdateEvent <sup>(1)</sup>
    ├── CategoryUpdateNameEvent
    └── CategoryUpdatePositionEvent
</pre>

### Store Channel Events

<pre>
GenericStoreChannelEvent
├── StoreChannelCreateEvent
├── StoreChannelDeleteEvent
└── GenericStoreChannelUpdateEvent <sup>(1)</sup>
    ├── StoreChannelUpdatePositionEvent
    └── StoreChannelUpdateNameEvent
</pre>

### Private Channel Events

Note: We don't use a generic event here because there are only 2 events.

<pre>
PrivateChannelCreateEvent
PrivateChannelDeleteEvent
</pre>

## Role Events

<pre>
GenericRoleEvent
├── RoleCreateEvent
├── RoleDeleteEvent
└── GenericRoleUpdateEvent <sup>(1)</sup>
    ├── RoleUpdateColorEvent
    ├── RoleUpdateHoistedEvent
    ├── RoleUpdateMentionableEvent
    ├── RoleUpdateNameEvent
    ├── RoleUpdatePermissionsEvent
    └── RoleUpdatePositionEvent
</pre>

## Emote Events

<pre>
GenericEmoteEvent
├── EmoteAddedEvent
├── EmoteRemovedEvent
└── GenericEmoteUpdateEvent <sup>(1)</sup>
    ├── EmoteUpdateNameEvent
    └── EmoteUpdateRolesEvent
</pre>

[^1]: This extends UpdateEvent<br>
[^2]: This event needs to be explicitly enabled in the JDABuilder/DefaultShardManagerBuilder<br>
[^3]: This extends GenericUserPresenceEvent
