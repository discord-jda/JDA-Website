# List of Events

All events mentioned in this list are a sub-type of [GenericEvent](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/events/GenericEvent.html)

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
├── SelfUpdateMFAEvent
├── SelfUpdateNameEvent
└── SelfUpdateVerifiedEvent
</pre>


## User Events

<pre>
GenericUserEvent
├── GenericUserUpdateEvent <sup>(1)</sup>
│   ├── UserUpdateOnlineStatusEvent <sup>(3)</sup>
|   ├── UserUpdateActivityOrderEvent <sup>(3)</sup>
│   ├── UserUpdateActivitiesEvent <sup>(3)</sup>
│   ├── UserUpdateAvatarEvent
│   ├── UserUpdateDiscriminatorEvent
│   ├── UserUpdateFlagsEvent
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
├── MessageReactionRemoveAllEvent
├── MessageReactionRemoveEmojiEvent
└── GenericMessageReactionEvent
    ├── MessageReactionAddEvent
    └── MessageReactionRemoveEvent
</pre>

- **MessageBulkDeleteEvent** is a special event and does not extend `GenericMessageEvent`!

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
├── GuildMemberRemoveEvent
├── GenericGuildInviteEvent
│   ├── GuildInviteCreateEvent
│   └── GuildInviteDeleteEvent
├── GenericPermissionOverrideEvent
│   ├── PermissionOverrideCreateEvent
│   ├── PermissionOverrideDeleteEvent
│   └── PermissionOverrideUpdateEvent
├── GenericStageInstanceEvent
│   ├── StageInstanceCreateEvent
│   ├── StageInstanceDeleteEvent
│   └── GenericStageInstanceUpdateEvent <sup>(1)</sup>
│       ├── StageInstanceUpdatePrivacyLevelEvent
|       └── StageInstanceUpdateTopicEvent
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
│   ├── GuildUpdateSplashEvent
│   ├── GuildUpdateSystemChannelEvent
│   ├── GuildUpdateBannerEvent
│   ├── GuildUpdateBoostCountEvent
│   ├── GuildUpdateBoostTierEvent
│   ├── GuildUpdateDescriptionEvent
│   ├── GuildUpdateMaxMembersEvent
│   ├── GuildUpdateMaxPresencesEvent
│   ├── GuildUpdateVanityCodeEvent
│   ├── GuildUpdateCommunityUpdatesChannelEvent
│   ├── GuildUpdateLocaleEvent
│   ├── GuildUpdateNSFWLevelEvent
│   ├── GuildUpdateRulesChannelEvent
│   └── GuildUpdateVerificationLevelEvent
├── GenericGuildMemberEvent
│   ├── GuildMemberJoinEvent
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
    ├── GuildVoiceRequestToSpeakEvent
    ├── GuildVoiceVideoEvent
    ├── GuildVoiceSuppressEvent
    └── GuildVoiceUpdateEvent <sup>(1)</sup>
        ├── GuildVoiceJoinEvent
        ├── GuildVoiceLeaveEvent
        └── GuildVoiceMoveEvent
</pre>

- **UnavailableGuildJoinedEvent** and **UnavailableGuildLeaveEvent** are special events and do not extend `GenericGuildEvent`!

## Channel Events

<pre>
GenericChannelEvent
├── ChannelCreateEvent
├── ChannelDeleteEvent
└── GenericChannelUpdateEvent <sup>(1)</sup>
    ├── ChannelUpdateArchivedEvent
    ├── ChannelUpdateArchiveTimestampEvent
    ├── ChannelUpdateAutoArchiveDurationEvent
    ├── ChannelUpdateBitrateEvent
    ├── ChannelUpdateInvitableEvent
    ├── ChannelUpdateLockedEvent
    ├── ChannelUpdateNameEvent
    ├── ChannelUpdateNSFWEvent
    ├── ChannelUpdateParentEvent
    ├── ChannelUpdatePositionEvent
    ├── ChannelUpdateRegionEvent
    ├── ChannelUpdateSlowmodeEvent
    ├── ChannelUpdateTopicEvent
    ├── ChannelUpdateTypeEvent
    └── ChannelUpdateUserLimitEvent
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
    ├── RoleUpdateIconEvent
    └── RoleUpdatePositionEvent
</pre>

## Emoji Events

<pre>
GenericEmojiEvent
├── EmojiAddedEvent
├── EmojiRemovedEvent
└── GenericEmojiUpdateEvent <sup>(1)</sup>
    ├── EmojiUpdateNameEvent
    └── EmojiUpdateRolesEvent
</pre>

## Sticker Events

<pre>
GenericGuildStickerEvent
├── GuildStickerAddedEvent
├── GuildStickerRemovedEvent
└── GenericGuildStickerUpdateEvent <sup>(1)</sup>
    ├── GuildStickerUpdateAvailableEvent
    ├── GuildStickerUpdateDescriptionEvent
    ├── GuildStickerUpdateNameEvent
    └── GuildStickerUpdateTagsEvent
</pre>

[^1]: This extends UpdateEvent<br>
[^2]: This event needs to be explicitly enabled in the JDABuilder/DefaultShardManagerBuilder<br>
[^3]: This extends GenericUserPresenceEvent
