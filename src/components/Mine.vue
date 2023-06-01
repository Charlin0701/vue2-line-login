<template>
	<div class="mine-container">
		<div class="content">
			{{ resp }}
		</div>

		<button @click="handleLogin">Login</button>
		<button @click="getUserInfo">User Info</button>
		<button @click="handleShare">Share</button>
	</div>
</template>

<script>
import qs from 'qs';
import axios from 'axios';

export default {
	data() {
		return {
			resp: null,
			// line的配置信息,将你申请好的应用信息填到此处
			config: {
				channel_id: '',
				channel_secret: ''
			},
			access_token: ''
		};
	},
	created() {
		// 当登录成功后回调会重新回到指定页面,所以我们在指定页面中判断是否是line登录

		// 由于line登录在回调地址会携带code和state参数,所以我们可以使用这个判断
		if (this.$route.query.code && this.$route.query.state) {
			console.log('line 登录');
			this.getLineToken();
		}
	},
	methods: {
		// 处理登录逻辑
		handleLogin() {
			const line_auth = 'https://access.line.me/oauth2/v2.1/authorize';
			const auth_params = {
				response_type: 'code',
				client_id: this.config.channel_id,
				redirect_uri: window.location.href, // 在LINE Developers Console上注册的回调 URL 的 URL 编码字符串。您可以添加任何查询参数。
				state: 'STATE', // 用于防止跨站点请求伪造的唯一字母数字字符串. 您的网络应用应为每个登录会话生成一个随机值。这不能是 URL 编码的字符串。
				scope: 'profile openid email' // 向用户请求的权限,查询范围可以看官网(https://developers.line.biz/en/docs/line-login/integrate-line-login/#scopes)
			};
			// 这里使用了qs处理参数
			const paramsString = qs.stringify(auth_params);
			window.location.href = `${line_auth}?${paramsString}`;
			console.log(`${line_auth}?${paramsString}`);
		},
		// 获取line的访问令牌
		async getLineToken() {
			const params = {
				grant_type: 'authorization_code', // 固定值
				code: this.$route.query.code, // 从 LINE 平台收到的授权码
				client_id: this.config.channel_id,
				client_secret: this.config.channel_secret,
				redirect_uri: window.location.origin + window.location.pathname // 与授权请求redirect_uri中指定的值相同
			};

			const resp = await axios({
				url: 'https://api.line.me/oauth2/v2.1/token',
				method: 'post',
				headers: {
					'Content-Type': 'application/x-www-form-urlencoded'
				},
				data: params
			});
			if (resp.status === 200) {
				// 请求成功
				this.resp = resp.data;
				this.access_token = resp.data.access_token;
			}
		},
		async getUserInfo() {
			if (!this.access_token) {
				alert('请先登录');
				return;
			}

			const resp = await axios({
				url: 'https://api.line.me/oauth2/v2.1/userinfo',
				headers: {
					Authorization: `Bearer ${this.access_token}`
				}
			});
			if (resp.status === 200) {
				// 请求成功
				this.resp = resp.data;
			}
		},
		// 分享
		handleShare() {
			const content = '这里是分享内容';
			window.open(`https://line.me/R/msg/text/?${content}`);
		}
	}
};
</script>

<style>
.content {
	width: 80%;
	min-width: 300px;
	height: 60%;
	min-height: 300px;
	background: lightblue;
	margin: 10px auto;
}

button {
	width: 100px;
	height: 30px;
	margin: 30px 20px;
}
</style>
