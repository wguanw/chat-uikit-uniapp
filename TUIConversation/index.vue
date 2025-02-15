<template>
    <scroll-view
      class="conversation-page"
      v-if="conversationList.length>0"
      scroll-y
      scroll-anchoring
      refresher-enabled
      scroll-with-animation
      :refresher-triggered="triggered"
      :refresher-threshold="80"
      refresher-background="#f9f9f9"
      @refresherrefresh="refresh"
    >
      <TUIConversationList
        :currentID="currrentConversationID"
        :conversationList="conversationList"
        @handleGotoItem="handleCurrrentConversation"
      />
    </scroll-view>
    <empty-page modeType="message" v-else></empty-page>
</template>
<script lang="ts">
import { defineComponent, reactive, toRefs, computed, onMounted } from "vue";
import { onNavigationBarButtonTap, onShow, onUnload } from "@dcloudio/uni-app";
import TUIConversationList from "./conversation-list";
import Dialog from "./components/dialog.vue";
import store from "../../../TUICore/store";
import { TUIConversationServer, TUIProfileServer } from "../../../TUICore/server";

const TUIConversation = defineComponent({
  name: "TUIConversation",
  components: {
    TUIConversationList,
    Dialog,
  },

  setup(props) {
    const timStore = store.state.timStore;
    uni.$TUIKit.TUIConversationServer = new TUIConversationServer();
    uni.$TUIKit.TUIProfileServer = new TUIProfileServer();
    // const { t } = uni.$TUIKit.config.i18n.useI18n();
    const data: any = reactive({
      triggered: true,
      conversationList: computed(() => timStore.conversationList),
      currrentConversationID: "",
      open: false,
      searchUserID: "",
      selectedList: [],
      searchUserList: [],
      step: 1,
      showDialog: false,
      item: {
        flow: "out",
        status: "success",
      },
      styleConfig: {
        width: "150px",
        // height: '160px',
        padding: "16px 16px 0px",
        top: "8px",
        right: "8px",
      },
      chatList: [
        {
          imgType: "SINGLE",
          type: uni.$TIM.TYPES.CONV_C2C,
          id: 1,
          content: "发起会话",
        },
        {
          imgType: "GROUP",
          type: uni.$TIM.TYPES.GRP_WORK,
          id: 1,
          content: "工作群",
        },
        {
          imgType: "GROUP",
          type: uni.$TIM.TYPES.GRP_PUBLIC,
          id: 2,
          content: "社交群",
        },
        {
          imgType: "GROUP",
          type: uni.$TIM.TYPES.GRP_MEETING,
          id: 3,
          content: "会议群",
        },
      ],
    });
  
    onUnload(() => {
     uni.$TUIKit.TUIConversationServer.destroyed();
    });
    
    onNavigationBarButtonTap(() => {
      data.showDialog = !data.showDialog;
    });
    onShow(() => {
      store.commit("timStore/setConversationID", "");
    });
    const refresh = () => {
      data.triggered = true;
			setTimeout(() => {
        data.triggered = false
			}, 600);
      setTimeout(() => {
			  // getConversationList();
      }, 1200);
		}
    const getConversationList = () => {
			uni.$TUIKit.getConversationList().then(imResponse => {
        this.conversationList = imResponse.data.conversationList
			});
		}
    // 切换当前会话
    const handleCurrrentConversation = (value: any) => {
      data.currrentConversationID = value.conversationID;
      store.commit("timStore/setConversationID", value.conversationID);
      //uni.$TUIKit.TUIChatServer.updateStore(value.conversationID)
      uni.navigateTo({
        url: `../TUIChat/index?conversationName=${handleItemName(value)}`,
      });
      uni.$TUIKit.TUIConversationServer.setMessageRead(value.conversationID);
      const curConversation = data.conversationList.filter((item: any) => {
        return item.conversationID === value.conversationID;
      });
      store.commit("timStore/setConversation", curConversation);
      uni.$TUIKit.TUIConversationServer.getConversationProfile(
        value.conversationID
      ).then((res: any) => {
        // 通知 TUIChat 关闭当前会话
        store.commit("timStore/setConversation", res.data.conversation);
        // uni.$TUIKit.getStore()['TUIChat'].conversation = res.data.conversation;
      });
    };

    // dialog 内部有效区域点击
    const handleContentClick = (item) => {
      data.showDialog = false;
      uni.navigateTo({
        url: `../TUIConversation/create?title=${item.content}&type=${item.type}`,
      });
    };
    // 关闭 dialog
    const handleClose = () => {
      data.showDialog = false;
    };
    const handleItemName = (item: any) => {
      let name = "";
      switch (item.type) {
        case uni.$TIM.TYPES.CONV_C2C:
          name = item?.userProfile.nick || item?.userProfile?.userID || "";
          break;
        case uni.$TIM.TYPES.CONV_GROUP:
          name = item.groupProfile.name || item?.groupProfile?.groupID || "";
          break;
        case uni.$TIM.TYPES.CONV_SYSTEM:
          name = "系统通知";
          break;
      }
      return name;
    };

    return {
      ...toRefs(data),
      refresh,
      getConversationList,
      handleCurrrentConversation,
      handleContentClick,
      handleItemName,
      handleClose,
    };
  },
});
export default TUIConversation;
</script>

<style lang="scss" scoped>
.conversation-page {
  height: 100vh;
  .create-group {
    font-weight: 800px;
    padding: 10px;
    font-size: 14px;
    text-align: center;
  }
}

.chat-container {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
  color: #444444;

  .item-icon {
    display: inline-block;
    width: 21px;
    height: 21px;
    margin-right: 12px;
  }
}
</style>
