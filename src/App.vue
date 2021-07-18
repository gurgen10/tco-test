<template>
  <div>
    <div>
      <b-navbar toggleable="lg" type="dark" class="tco-navbar">
        <b-navbar-brand href="#">{{ page_title }}</b-navbar-brand>

        <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

        <b-collapse id="nav-collapse" is-nav>

          <!-- Right aligned nav items -->
          <div
            class="plan"
            :class="{active: plan.active, [plan.name.toLowerCase()]: true}"
            v-for="plan in plans"
            :style="{
              'background-image': 
              `url(${require('@/assets/img/'+ (plan.active ? plan.name.toLowerCase()+ '_active' : plan.name.toLowerCase() )+'.png')})`
            }"
            :key="plan.name">
            <span>{{plan.name}}</span>
          </div>
          <b-navbar-nav class="ml-auto">
            <b-nav-form>
              <b-button size="sm" class="upgrade-now my-2 my-sm-0"
                >Upgrade Now</b-button
              >
            </b-nav-form>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
      <div>
        <div>
          <b-tabs content-class="mt-3" v-model="tabIndex" class="tco-tabs">
            <b-tab
              v-for="(tab, index) in tabs"
              :key="index"
              :title="tab.title"
              active
              class="tco-tab"
            >
              <div v-if="tab.data.systems && tab.data.systems.length">
                <div class="system-header">Systems</div>
                <div class="fold-table accordion" role="tablist">
                  <table >
                    <thead>
                      <tr>
                        <th>System Name</th>
                        <th>ID</th>
                        <th>Created date</th>
                        <th>Licence</th>
                      </tr>
                    </thead>
                    <tbody>
                    <template v-for="row in tab.data.systems" >
                      <tr
                        :key="row.id"
                        @click="activeSystemId = row.id"
                        class="view"
                        v-b-toggle="'accordion-' + row.id"
                      >
                        <td class="table-data-td">
                          <i class="showMore fa fa-caret-right"></i>
                          <span>{{ row.system_name }}</span></td>
                        <td class="table-data-td">{{ row.id }}</td>
                        <td class="table-data-td">{{ row.created_date }}</td>
                        <td class="table-data-td">{{ row.active_licenses }}</td>
                        <td class="table-data-td buttons-td">
                          <b-button class="margin-right-15" variant="success">Add License</b-button>
                          <b-button class="margin-right-15" variant="danger">Remove</b-button>
                        </td>
                      </tr>
                      <tr class="fold" :key="row.id + 'g'">
                        <td colspan="7">
                        <b-collapse
                          :id="'accordion-' + row.id"
                          accordion="my-accordion"
                          role="tabpanel"
                          class="sub-table-container"
                        >
                        <div class="">
                          <div
                            class="table-lift"
                            v-for="(subRow, indx) in tab.data.subsystems.filter((s) => s.system_id === activeSystemId)"
                            :key="indx"
                          ></div>
                        </div>
                        <table class="sub-table">
                          <thead>
                            <tr class="sub-table-row-header">
                              <th>System Name</th>
                              <th>ID</th>
                            </tr>
                          </thead>
                          <tbody>
                            <tr
                            class="sub-table-row"
                              v-for="(
                                subRow, indx
                              ) in tab.data.subsystems.filter(
                                (s) => s.system_id === activeSystemId
                              )"
                              :key="indx"
                            >
                              <td data-th="Company name">
                                {{ subRow.licenses }}
                              </td>
                              <td data-th="Customer no">
                                {{ subRow.expires }}
                              </td>
                              <td data-th="Customer no">
                                Download
                              </td>
                            </tr>
                          </tbody>
                        </table>
                        </b-collapse>
                        </td>
                      </tr>
                    </template>
                    </tbody>
                  </table>
                </div>
              </div>
            </b-tab>
          </b-tabs>
          <div>
            <div class="system-header">Add System</div>
          </div>
          <div ref="prev" ></div>
          <b-form-group id="fileInput" >
            <b-form-file 
              v-model="files"
              ref="file-input"
              @change="uploadFile"
              :state="fileSize < 5"
              required
              class="uploaded-files"
              accept="image/jpeg, image/png"
              placeholder=""
              multiple
              >
            </b-form-file>
            <div>
            <div ref="preview" class="uploaded-images"></div>
            <div class="input-placeholder">
                <div class="arrow-down-container"><i class="fa fa-arrow-down"></i></div>
                <div class="placeholder">
                  <span @click="openFileInput">Choose a file</span> to upload or drag it here ? (jpg, png, max size 5mb)
                </div>
              </div>
            </div>
            <b-alert :show="fileSize > 5120" variant="danger">Your uploaded {{fileSize}}mb: max siz is 5mb</b-alert>
          </b-form-group>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import jsonData from "@/assets/tast_data.json";
export default {
  name: "App",
  data: () => ({
    page_title: jsonData.page_title,
    plans: jsonData.plans,
    tabs: jsonData.tabs,
    opened: [],
    activeSystemId: null,
    tabIndex: 0,
    files: null,
    previewImg: [],
    fileObj: [],
    fileSize: 0
  }),
  watch: {
    files(val) {
      if(this.files.length) {
        document.querySelector('.input-placeholder').style.display = 'none';
      }
      this.$refs.preview.innerHTML = '';
      this.fileSize = 0
      Array.from(this.files).forEach((file, idx) => {
        this.fileSize += (file.size/1048576).toFixed(1);
        var image = file
        var reader = new FileReader();
        reader.onload = (file) => {
          var img = new Image();
          img.src = file.target.result;
          const timesCircle = document.createElement('i')
          timesCircle.className = 'faIcon fa fa-times-circle';
          timesCircle.setAttribute('data-name', this.files[idx].name)
          timesCircle.setAttribute('data-idx', idx)
          const _files = this.files;
          const fileInput = this.$refs['file-input']
          timesCircle.addEventListener('click', function(e) {
            Array.prototype.splice.call(_files, idx, 1)
            this.parentNode.remove()
            if(!_files.length) {
              fileInput.reset()
            }
          })
          const imgDiv = document.createElement('div')
          imgDiv.className = 'img-div'
          imgDiv.appendChild(img);
          imgDiv.appendChild(timesCircle);
          this.$refs.preview.appendChild(imgDiv);
        }
        reader.readAsDataURL(image);
        document.querySelector('.d-block.form-file-text').innerHTML = '';
      })
    }
  },
  mounted() {
    
  },
  methods: {
    uploadFile() {
      document.querySelector('.d-block.form-file-text').innerHTML = '';
    },
    openFileInput () {
      this.$refs['file-input'].$el.click();
    },
    toggle(id) {
      const index = this.opened.indexOf(id);
      if (index > -1) {
        this.opened.splice(index, 1);
      } else {
        this.opened.push(id);
      }
    },
  },
};
</script>

<style lang="scss">
@import url("https://netdna.bootstrapcdn.com/font-awesome/4.0.3/css/font-awesome.css");
@import url(https://fonts.googleapis.com/css?family=Oswald:700);
@import url(https://fonts.googleapis.com/css?family=Droid+Serif);

// Fonts
$roboto: "Roboto";
$icon: fontawesome;

// Table
$bgColor:#1F2224;
$bgFoldColor:#1A1C1E;
$textColor: #D6DDDE;
$activeTextColor: #FFFFFF;
$borderColor: rgba(139, 144, 145, 0.25);
$activeBorderColor: #18A7FF;

// Navbar
body {
  background-color: $bgColor !important;
  color: $textColor;
  font-family: $roboto;
  padding: 35px 33px;

}
.margin-right-15 {
  margin-right: 15px;
}
.tco-navbar {
  .navbar-brand {
    font-family: Roboto;
    font-style: normal;
    font-weight: normal;
    font-size: 28px;
    line-height: 33px;
    color: $textColor !important;
  }
}

.navbar-collapse.collapse {
  justify-content: flex-end;
}
.plan {
  position: relative;
  width: 48px;
  height: 48px;
  margin-right: 36px;
  background-repeat: no-repeat;
  background-size: contain;
  cursor: pointer;
  span {
    position: absolute;
    text-align: center;
    width: 100%;
    top: 50%;
    font-family: Roboto;
    font-style: normal;
    font-weight: normal;
    font-size: 14px;
    line-height: 16px;
    text-align: center;
    transform: translateY(-50%);
    color: $textColor;
    display: inline-block;
    width: 100%;
    padding: 2px;
    text-transform: uppercase;
    padding: 2px;
    background: linear-gradient(0deg, #131718, #131718), linear-gradient(0deg, #131718, #131718), linear-gradient(0deg, #131718, #131718), linear-gradient(0deg, #131718, #131718), #131718;
    opacity: 0.25;
  }
  &.active {
    span {
      background: linear-gradient(0deg, #131718, #131718), linear-gradient(0deg, #131718, #131718), linear-gradient(0deg, #131718, #131718), linear-gradient(0deg, #131718, #131718), #131718;
      opacity: 1;
    }
  }
}
.upgrade-now {
  background-color: $activeBorderColor !important;
  padding: 8px 20px !important;
}

// content
.system-header {
  font-family: $roboto;
  font-style: normal;
  font-weight: normal;
  font-size: 16px;
  line-height: 19px;
  color: $activeTextColor;
  margin-bottom: 24px;
  margin-top: 43px;
}

.nav-tabs {
  border-bottom: 1px solid $borderColor;
  margin-top: 128px;
  .nav-item {
    .nav-link {
      background: $bgColor;
      color: $textColor;
      border: none;
      &.active {
        color: $activeTextColor;
        background: $bgColor;
        padding-bottom: 22px;
        opacity: 1;
        border-bottom: 2px solid $activeBorderColor;
      }
    }
  }
}

// Table
table {
  position: relative;
  width: 100%;
  font-family: $roboto;
  th {
    text-align: left;
    vertical-align: bottom;
    border: none;
    color: $textColor;
  }
  td {
    color: $activeTextColor;
    padding: 12px 15px;
  }
  .buttons-td {
    text-align: right;
  }
}

// fold table
table.fold-table {
  tr {
      color: $bgFoldColor;
      transition: all 0.3s ease;
    td,
    th {
      cursor: pointer;
      padding-bottom: 8px;
    } 
  }
  tr.fold {
    padding: 0;
    display: none;
    &.open {
      display: table-row;
    }
  }
}
.view {
  border-bottom: 8px solid $bgColor;
  background-color: $bgFoldColor;
  .showMore {
    margin-right: 18px;
    margin-left: 16px;
  }
  &.not-collapsed {
    border-bottom: none;
    .showMore {
      transform: rotate(90deg);
      transition: all 0.3s ease;
    }
  }
}
.sub-table {
   background-color: $bgColor;
  .sub-table-row {
    border-bottom: 8px solid $bgColor;
    td {
      background-color: $bgFoldColor;
    }
    
  }
}
.sub-table-container {
  display: flex;
}
.table-lift {
  margin-top: -5px;
  margin-left: 52px;
  height: 56px;
  width: 30px;
  border-left: 8px solid $bgFoldColor;
    border-bottom: 8px solid $bgFoldColor;
}
.fold {
  > td {
    padding: 0;
  }
}
.table-data-td {
  line-height: 0;
  padding: 12px 15px;
}
.uploaded-files {
  width: 100%;
  height: 200px;
  color: #fff;
  .custom-file,
  .custom-file-input {
    height: 100px;
    
  }
  input[type='file'] {
    display: none;
  }
  .custom-file-label {
    border: 0;
    border: 1px dashed $borderColor;
    height: 100%;
    width: 100%;
    line-height: 90px;
    text-align: center;
    color: #FFF;
    padding: 0;
    z-index: 100;
    background-color: transparent;
    cursor: pointer;
  }
}
.d-block.form-file-text {
  color: transparent;
}
.uploaded-images {
  position: relative;
  top: -180px;
  left: 20px;
  padding:24px;
  .img-div {
    position: relative;
    display: inline-block;
    &:not(:first-child) {
      margin-left: 21px;
    }
    .faIcon {
      position: absolute;
      top: -10px;
      right: -10px;
      z-index: 10;
      font-size: 20px;
      cursor: pointer;
      color: red !important;
    }
    img {
      width: 100px;
    }
  }
}
.fileInput {
  position: relative;
}
.input-placeholder {
  text-align: center;
  position: relative;
  color: #FFF;
  top: -200px;
  display: inline-block;
  left: 50%;
  transform: translateX(-50%);
  .arrow-down-container {
    font-family: $icon;
    width: 40px;
    height: 40px;
    display: inline-flex;
    justify-content: center;
    align-items: center;
    border: 2px solid #FFF;
    margin-bottom: 28px;
  }
  .placeholder {
    font-family: Roboto;
    font-style: normal;
    font-weight: normal;
    font-size: 16px;
    line-height: 19px;
    max-width: 270px;
    span {
      color: $activeBorderColor;
      font-family: Roboto;
      font-style: normal;
      font-weight: normal;
      font-size: 16px;
      line-height: 19px;
      cursor: pointer;
    }
  }
}
</style>
