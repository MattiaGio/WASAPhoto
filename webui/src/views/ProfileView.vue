<script>
import LogModal from "../components/Logmodal.vue";
import SuccessMsg from "../components/SuccessMsg.vue"

export default {
    components: { LogModal, SuccessMsg },
    data: function () {
        return {
            errormsg: null,
            successmsg: null,
			detailedmsg: null,
            username: localStorage.getItem('username'),
            token: localStorage.getItem('token'),
            newUsername: "",
            profile: {
                requestId: 0,
                id: 0,
                username: "",
                followersCount: 0,
                followingCount: 0,
                photoCount: 0,
            },
            photoList: {
                requestUser: 0,
                identifier: 0,
                photos: [
                    {
                        id: 0,
                        userId: 0,
                        file: "",
                        date: "",
                        likesCount: 0,
                        commentsCount: 0,
                        likeStatus: null,
                        comment: "",
                    }
                ],
            },
            user: {
                id: 0,
                username: "",
            },
            comment: "",
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
        }
    },
    methods: {

        async refresh() {
            await this.userProfile();
            await this.userPhotos();
        },
        async uploadFile() {
			this.images = this.$refs.file.files[0]
			/*this.submitFile*/
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
                    this.errormsg = null;
                    this.refresh();
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
        async userProfile() {
            try {
                let response = await this.$axios.get("/users/" + this.username + "/profile", {
                    headers: {
                        Authorization: "Bearer " + localStorage.getItem("token")
                    }
                })
                this.profile = response.data
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
        async userPhotos() {
            try {
                let response = await this.$axios.get("/users/" + this.username + "/photo", {
                    headers: {
                        Authorization: "Bearer " + localStorage.getItem("token")
                    }
                })
                this.photoList = response.data
                for (let i = 0; i < this.photoList.photos.length; i++) {
                    this.photoList.photos[i].file = 'data:image/*;base64,' + this.photoList.photos[i].file
                }
                
            } catch (e) {
                if (e.response && e.response.status === 400) {
                    this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
                    this.detailedmsg = null;
                } else if (e.response && e.response.status === 500) {
                    this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
                    this.detailedmsg = e.toString();
                } else {
                    this.errormsg = "You haven't posted any photos yet. Go to your profile and upload one!";
                    this.detailedmsg = null;
                }
            }
        },
        async deletePhoto(photoid) {
            try {
                let response = await this.$axios.delete("/users/" + this.username + "/photo/" + photoid, {
                    headers: {
                        Authorization: "Bearer " + localStorage.getItem("token")
                    }
                })
                this.refresh();
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
        async changeName() {
            if (this.newUsername == "") {
                this.errormsg = "Emtpy username field."
            } else {
                try {
                    let response = await this.$axios.put("/user/" + this.username + "/setusername", { username: this.newUsername }, {
                        headers: {
                            Authorization: "Bearer " + localStorage.getItem("token")
                        }
                    })
                    this.user = response.data
                    localStorage.setItem("username", this.user.username);
                    this.profile.username = this.user.username;
                    this.username = this.user.username;
                    this.$router.push({ path: '/users/' + this.user.username + '/profile' })
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
        async sendComment(username, photoid,comment) {
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
        async UploadShortucut() {
			this.$router.push({ path: '/users/' + this.username + '/upload' })
		},
    },
    mounted() {
        this.userProfile()
        this.userPhotos()
    }
}
</script>

<template>
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
    <Router-link to="/" class="nav-link d-flex align-items-center" @click="doLogout">
        <div style="border: 2px solid red; padding: 5px; border-radius: 5px;">
            <span style="font-size: 20px; color: red;">Logout</span>
        </div>
    </Router-link>
</li>
        </ul>
    </div>
</nav>


    <div class="d-flex justify-content-center align-items-center flex-column">
        <h1 class="h2">Welcome to your profile <span style="color: #8A3FFC;">{{ profile.username }}</span></h1>

        <div class="p-4 text-black">
            <div class="d-flex justify-content-end text-center py-1">
                <div>
                    <p class="mb-1 h5 text-primary">{{ profile.followersCount }}</p>
                    <p class="small text-muted mb-0">Followers</p>
                </div>
            <div class="px-3">
                <p class="mb-1 h5 text-success">{{ profile.followingCount }}</p>
                <p class="small text-muted mb-0">Followings</p>
            </div>
            <div>
                <p class="mb-1 h5 text-purple" style="color: #8A3FFC;">{{ profile.photoCount }}</p>
                <p class="small text-muted mb-0" >Photos</p>
            </div>

            </div>
        </div>
    </div>

    <div class="mb-3" style="text-align: left; display: flex; justify-content: space-between;">
    <p class="small mb-0" style="color: black; font-size: 26px;">Change ur username <span style="color: #8A3FFC;">here</span></p>
    <p class="small mb-0" style="color: black; font-size: 26px;"><span style="color: #0d6efd;">Upload</span> ur photo <span style="color: #8A3FFC;">here</span></p>
</div>

<div class="mb-3" style="text-align: left; display: flex; justify-content: space-between;">
    
    <div class="input-group" style="max-width: 550px; display: flex; justify-content: space-between;">
        <input type="text" id="newUsername" v-model="newUsername" class="form-control text-center"
            placeholder="Insert a new username for your profile..." aria-label="Recipient's username"
            aria-describedby="basic-addon2">
        <button class="btn btn-primary" type="button" @click="changeName">Change username</button>
    </div>
    <div class="input-group" style="max-width: 420px; display: flex; justify-content: space-between;">
        <input type="file" accept="image/*" class="btn btn-outline-primary" @change="uploadFile" ref="file">
        <button class="btn btn-success" style="background-color: #8A3FFC;" @click="submitFile">Upload</button>
    </div>
</div>

    <div
        class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
    </div>

    <ErrorMsg v-if="errormsg" :msg="errormsg"></ErrorMsg>
    <SuccessMsg v-if="successmsg" :msg="successmsg"></SuccessMsg>

    <LogModal id="logviewer" :log="photoComments" :token="token"></LogModal>

    <div class="row">
        <div class="col-md-4" v-for="photo in photoList.photos" :key="photo.id">
            <div class="card mb-4 shadow-sm">
                <img class="card-img-top" :src=photo.file alt="Card image cap">
                <div class="card-body">
                    <RouterLink :to="'/users/' + profile.username + '/profile'" class="nav-link">
                        <button type="button" class="btn btn-outline-primary">{{ profile.username }}</button>
                    </RouterLink>
                    <div
                        class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
                    </div>
                    <div class="d-flex justify-content-between align-items-center">
                        <p class="card-text">Likes : {{ photo.likesCount }}</p>
                    </div>
                    <div class="d-flex justify-content-between align-items-center">
                        <p class="card-text">Comments : {{ photo.commentsCount }}</p>
                    </div>
                    <p class="card-text">Photo uploaded on {{ photo.date }}</p>

                    <div class="input-group mb-3">
                        <input type="text" id="comment" v-model="photo.comment" class="form-control" placeholder="Comment!"
                          aria-describedby="basic-addon2">
                        <div class="input-group-append">
                            <button class="btn btn-primary" type="button"
                                @click="sendComment(username, photo.id, photo.comment)">Send</button>
                        </div>
                    </div>

                    <div class="d-flex justify-content-between align-items-center">
                        <div class="btn-group">
                            <button type="button" class="btn btn-dark" @click="openLog(username, photo.id)">Comments</button>
                            <button type="button" v-if="photo.likeStatus == false" class="btn btn-primary"
                                @click="likePhoto(username, photo.id)">Like</button>
                            <button type="button" v-if="photo.likeStatus == true" class="btn btn-danger"
                                @click="deleteLike(username, photo.id)">Unlike</button>
                            <button type="button" class="btn btn-sm btn btn-outline-danger"
                                @click="deletePhoto(photo.id)">Delete </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>


</template>

<style>
</style>
