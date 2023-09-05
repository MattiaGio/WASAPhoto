<script>
import LogModal from "../components/Logmodal.vue";
import SuccessMsg from "../components/SuccessMsg.vue";

export default {
	components: { LogModal, SuccessMsg },
	data: function () {
		return {
			errormsg: null,
			successmsg: null,
			detailedmsg: null,
			username: localStorage.getItem('username'),
			token: localStorage.getItem('token'),
			loading: false,
			some_data: null,
			images: null,
			image: null,
			clear: null,
			photoComments: {
				requestIdentifier: 0,
				photoIdentifier: 0,
				identifier: 0,
				comments: [
					{
						id: 0,
						userId: 0,
						photoId: 0,
						photoOwner: 0,
						ownerUsername: "",
						username: "",
						content: "",
					}
				],
			},
			comment: "",
			stream: {
				identifier: 0,
				photoStream: [
					{
						id: 0,
						userId: 0,
						username: "",
						file: "",
						date: "",
						likeCount: 0,
						commentCount: 0,
						comment: "",
						likeStatus: null,
					}
				],
			},
			searchUserUsername: "",
			like: {
				likeId: 0,
				identifier: 0,
				photoIdentifier: 0,
				photoOwner: 0,
			},
			profile: {
				requestId: 0,
				id: 0,
				username: "",
				followersCount: 0,
				followingCount: 0,
				photoCount: 0,
				followStatus: null,
				banStatus: null,
			},
		}
	},
	methods: {
		async refresh() {
			this.getStream()
		},
		async uploadFile() {
			this.images = this.$refs.file.files[0]
			this.submitFile
		},
		async submitFile() {
			if (this.images === null) {
				this.errormsg = "Please select a file to upload."
			} else {
				try {
					let response = await this.$axios.put("/users/" + this.username + "/photo/" + Math.floor(Math.random() * 10000), this.images, {
						headers: {
							Authorization: "Bearer " + localStorage.getItem("token")
						}
					})
					this.profile = response.data
					this.successmsg = "Photo uploaded successfully."
				} catch (e) {
					if (e.response && e.response.status === 400) {
						this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
						this.detailedmsg = null;
					} else if (e.response && e.response.status === 500) {
						this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
						this.detailedmsg = e.toString();
					} else {
						this.errormsg = e.toString();
						this.detailedmsg = null;
					}
				}
			}
		},
		async getStream() {
			try {
				let response = await this.$axios.get("/user/" + this.username + "/stream", {
					headers: {
						Authorization: "Bearer " + localStorage.getItem("token")
					}
				})
				this.stream = response.data
				for (let i = 0; i < this.stream.photoStream.length; i++) {
					this.stream.photoStream[i].file = 'data:image/*;base64,' + this.stream.photoStream[i].file
				}
			} catch (e) {
				if (e.response && e.response.status === 400) {
					this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
					this.detailedmsg = null;
				} else if (e.response && e.response.status === 500) {
					this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
					this.detailedmsg = e.toString();
				} else {
					this.errormsg = "You are not following anyone yet. Follow someone to see their photos.";
					this.detailedmsg = null;
				}
			}
		},
		async SearchUser() {
			if (this.searchUserUsername === this.username) {
				this.errormsg = "You can't search yourself."
			} else if (this.searchUserUsername === "") {
				this.errormsg = "Emtpy username field."
			} else {
				try {
					let response = await this.$axios.get("users/" + this.searchUserUsername + "/profile", {
						headers: {
							Authorization: "Bearer " + localStorage.getItem("token")
						}
					})
					this.profile = response.data
					this.$router.push({ path: '/users/' + this.searchUserUsername + '/view' })
				} catch (e) {
					if (e.response && e.response.status === 400) {
						this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
						this.detailedmsg = null;
					} else if (e.response && e.response.status === 500) {
						this.errormsg = "User does not exist on WASAPhoto.";
						this.detailedmsg = e.toString();
					} else {
						this.errormsg = e.toString();
						this.detailedmsg = null;
					}
				}
			}
		},
		async sendComment(username, photoid, comment) {
			if (comment === "") {
				this.errormsg = "Emtpy comment field."
			} else {
				try {
					let response = await this.$axios.put("/users/" + username + "/photo/" + photoid + "/comment/" + Math.floor(Math.random() * 10000), { content: comment }, {
						headers: {
							Authorization: "Bearer " + localStorage.getItem("token")
						}
					})
					this.clear = response.data
					this.refresh()
				} catch (e) {
					if (e.response && e.response.status === 400) {
						this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
						this.detailedmsg = null;
					} else if (e.response && e.response.status === 500) {
						this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
						this.detailedmsg = e.toString();
					} else {
						this.errormsg = e.toString();
						this.detailedmsg = null;
					}
				}
			}
		},
		async openLog(username, photoid) {
			try {
				let response = await this.$axios.get("/users/" + username + "/photo/" + photoid + "/comment", {
					headers: {
						Authorization: "Bearer " + localStorage.getItem("token")
					}
				})
				this.photoComments = response.data;
				const modal = new bootstrap.Modal(document.getElementById('logviewer'));
				modal.show();
			} catch (e) {
				if (e.response && e.response.status === 400) {
					this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
					this.detailedmsg = null;
				} else if (e.response && e.response.status === 500) {
					this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
					this.detailedmsg = e.toString();
				} else {
					this.errormsg = e.toString();
					this.detailedmsg = null;
				}
			}
		},
		async likePhoto(username, id) {
			try {
				let response = await this.$axios.put("/users/" + username + "/photo/" + id + "/like/" + Math.floor(Math.random() * 10000), {}, {
					headers: {
						Authorization: "Bearer " + localStorage.getItem("token")
					}
				})
				this.clear = response.data
				this.refresh()
			} catch (e) {
				if (e.response && e.response.status === 400) {
					this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
					this.detailedmsg = null;
				} else if (e.response && e.response.status === 500) {
					this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
					this.detailedmsg = e.toString();
				} else {
					this.errormsg = e.toString();
					this.detailedmsg = null;
				}
			}
		},
		async deleteLike(username, id) {
			try {
				let response = await this.$axios.get("/users/" + username + "/photo/" + id + "/like", {
					headers: {
						Authorization: "Bearer " + localStorage.getItem("token")
					}
				})
				this.like = response.data
			} catch (e) {
				if (e.response && e.response.status === 400) {
					this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
					this.detailedmsg = null;
				} else if (e.response && e.response.status === 500) {
					this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
					this.detailedmsg = e.toString();
				} else {
					this.errormsg = e.toString();
					this.detailedmsg = null;
				}
			}

			try {
				let response = await this.$axios.delete("/users/" + username + "/photo/" + id + "/like/" + this.like.likeId, {
					headers: {
						Authorization: "Bearer " + localStorage.getItem("token")
					}
				})
				this.clear = response.data
				this.refresh()
			} catch (e) {
				if (e.response && e.response.status === 400) {
					this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
					this.detailedmsg = null;
				} else if (e.response && e.response.status === 500) {
					this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
					this.detailedmsg = e.toString();
				} else {
					this.errormsg = e.toString();
					this.detailedmsg = null;
				}
			}
		},
		async doLogout() {
			localStorage.removeItem("token")
			localStorage.removeItem("username")
			this.$router.push({ path: '/' })
		},
		async ViewProfile() {
			this.$router.push({ path: '/users/' + this.username + '/profile' })
		},
		async UploadShortucut() {
			this.$router.push({ path: '/users/' + this.username + '/upload' })
		},
	},
	mounted() {
		this.getStream()
	}
}
</script>



	<template>
		<div>
			<nav id="sidebarMenu" class="col-md-2 d-md-block bg-light sidebar collapse">
				<div class="position-sticky pt-3 sidebar-sticky">
    				<h6 class="sidebar-heading d-flex justify-content-center align-items-center px-2 mt-3 mb-1 text-muted text-uppercase" style="font-size: 25px;" >
        				<span style="text-decoration: underline;">General</span>
					</h6>
        <ul class="nav flex-column">
            <li class="nav-item">
                <RouterLink to="/session" class="nav-link d-flex align-items-center">
                    <svg class="feather">
                        <use href="/feather-sprite-v4.29.0.svg#home" />
                    </svg>
                    <span style="font-size: 20px;">Home</span>
                </RouterLink>
            </li>
            <li class="nav-item">
                <RouterLink to="`/users/${username}/upload`" class="nav-link d-flex align-items-center" @click="UploadShortucut"> 
                    <svg class="feather">
                        <use href="/feather-sprite-v4.29.0.svg#upload" />
                    </svg>
                    <span style="font-size: 14px;">Upload</span>
                </RouterLink>
            </li>
            <li class="nav-item">
                <Router-link :to="`/users/${username}/profile`" class="nav-link d-flex align-items-center" @click="ViewProfile">
                    <svg class="feather">
                        <use href="/feather-sprite-v4.29.0.svg#user" />
                    </svg>
                    <span style="font-size: 14px;">Profile</span>
                </Router-link>
            </li>
            <li class="nav-item">
    <Router-link to="/" class="nav-link d-flex align-items-center" @click="doLogout">
        <div style="border: 2px solid red; padding: 5px; border-radius: 5px;">
            <span style="font-size: 20px; color: red;">Logout</span>
        </div>
    </Router-link>
</li>
        </ul>
    </div>
</nav>




		<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
			<h1 class="h2" style="margin-left: 590px;">Welcome back <RouterLink :to="'/users/' + this.username + '/profile'" style="color: #8A3FFC;">{{ this.username }}</RouterLink></h1>
			<div class="btn-group">
				<button class="btn btn-primary dropdown-toggle" type="button" id="dropdownMenuButton" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
					Menu
				</button>
		
					<div class="dropdown-menu" aria-labelledby="dropdownMenuButton">
					<button class="dropdown-item" @click="ViewProfile">Profile</button>
					<button class="dropdown-item" @click="doLogout">Logout</button>
				</div>
			</div>
		</div>



		<div class="d-flex flex-column align-items-center justify-content-center" style="min-height: 10vh; background-color: #f0f0f0;">
   	 		<p class="h4">Search for your friends on WASAPhoto</p>
    		<div class="input-group mb-3" style="max-width: 450px; position: relative;">
        		<input type="text" id="searchUserUsername" v-model="searchUserUsername" class="form-control"
            		placeholder=" " aria-label="Recipient's username"
            		aria-describedby="basic-addon2">
        		<label for="searchUserUsername" class="position-absolute text-muted" style="top: 50%; left: 50%; transform: translate(-50%, -50%);" v-show="searchUserUsername === ''">Search a user in WASAPhoto.</label>
        		<div class="input-group-append">
            		<button class="btn btn-primary" type="button" @click="SearchUser">Search</button>
        		</div>
    		</div>
		</div>





		<div
			class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
		</div>

		<ErrorMsg v-if="errormsg" :msg="errormsg"></ErrorMsg>
		<SuccessMsg v-if="successmsg" :msg="successmsg"></SuccessMsg>

		<LogModal id="logviewer" :log="photoComments" :token="token"></LogModal>


	<div class="container mt-4">
  		<div class="row">
    		<div class="col-md-4" v-for="photo in stream.photoStream" :key="photo.id">
    		  <div class="card mb-4 shadow-sm">
     		   <img class="card-img-top" :src="photo.file" alt="Card image">
      			  <div class="card-body">
      			    <router-link :to="'/users/' + photo.username + '/view'" class="text-decoration-none">
       			     <h5 class="card-title">{{ photo.username }}</h5>
          			</router-link>
          <hr>
          <p class="card-text">Likes: {{ photo.likeCount }}</p>
          <p class="card-text">Comments: {{ photo.commentCount }}</p>
          <p class="card-text">Uploaded on: {{ photo.date }}</p>

          <div class="input-group mb-3">
            <input type="text" id="comment" v-model="photo.comment" class="form-control"
              placeholder="Add a comment" aria-label="Add a comment" aria-describedby="basic-addon2">
            <div class="input-group-append">
              <button class="btn btn-primary" type="button"
                @click="sendComment(photo.username, photo.id, photo.comment)">Send</button>
            </div>
          </div>

          <div class="btn-group" role="group">
            <button type="button" class="btn btn-outline-dark"
              @click="openLog(photo.username, photo.id)">View Comments</button>
            <button type="button" v-if="photo.likeStatus" class="btn btn-danger"
              @click="deleteLike(photo.username, photo.id)">Unlike</button>
            <button type="button" v-else class="btn btn-primary"
              @click="likePhoto(photo.username, photo.id)">Like</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>



	</div>
</template>

<style>

</style>
