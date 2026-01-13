#ci/cd

**Overall Workflow**
![image.png](https://codahosted.io/docs/g1l4X1Btfi/blobs/bl-6ln3M6scC-/883c90943a462074921583fe899e557f9e9bd490c80ecbc7f82a1b35d145f123589c973da3c15c6575b0eb43d71a99c8c11119e1c1f1a4d2023d241488a166597dee00adec64b804c398dd7131a3b1a476ca52332273abd5ccf82f9b020eb45f8a34be6e)
Setup Environment
ให้ setup ใน github workflow secrets ดังนี้

```
GH_PAT: ${{ secrets.GH_PAT }}

FIREBASE_DISTRIBUTION_IOS_APP_ID: ${{ secrets.FIREBASE_DISTRIBUTION_IOS_APP_ID }}

FIREBASE_DISTRIBUTION_ANDROID_APP_ID: ${{ secrets.FIREBASE_DISTRIBUTION_ANDROID_APP_ID }}

MATCH_KEYCHAIN_PASSWORD: ${{ secrets.MATCH_KEYCHAIN_PASSWORD }}

MATCH_GIT_BASIC_AUTHORIZATION: ${{ secrets.GH_PAT}}

ASC_KEY_ID: ${{ secrets.ASC_KEY_ID}}

ASC_ISSUER_ID: ${{ secrets.ASC_ISSUER_ID}}

ASC_KEY_P8: ${{ secrets.ASC_KEY_P8}}

FIREBASE_DISTRIBUTION_GOOGLE_SECREAT: ${{ secrets.FIREBASE_DISTRIBUTION_GOOGLE_SECREAT }}

GHA_PUBLISH_JSON_STAGING: ${{ secrets.GHA_PUBLISH_JSON_STAGING }}

KEYSTORE: ${{ secrets.KEYSTORE }}

KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}

KEYSTORE_KEY_ALIAS: ${{ secrets.KEYSTORE_KEY_ALIAS }}

KEYSTORE_KEY_PASSWORD: ${{ secrets.KEYSTORE_KEY_PASSWORD }}

ENV: ${{ secrets.ENV_STAGING }} --> เปล่ี่ยนเป็น ENV_PROD ถ้าจะ build ด้วย env prod
```

สำหรับ `ASC_KEY_P8` `KEY_STORE` ให้แปลงเป็น base64 ด้วยคำสั่ง
`base64 -i <ไฟล์ ASC หรือ KEYSTORE> | pbcopy` ก่อน ค่อยเอาไปใส่ใน github secrets

`MATCH_GIT_BASIC_AUTHORIZATION` ก็ให้แปลงเป็น base64 ก่อนใส่ด้วย ดังนี้
`echo -n your_github_username:your_personal_access_token | base64`

**_requirement_**

1. ในที่นี้จะใช้ ruby version 3.3.5 ดังนั้นถ้าอยากให้เป๊ะ โหลด chruby มาเปลี่ยน version ruby ในเครื่องก่อนโหลดพวก bundler (เอาไว้ใช้fastlane)
2. flutter ตอนนี้ใช้ version `3.38.4` สำหรับ b2c และ b2b

_ข้อควรระวัง_

1. เวลา run workflow ให้ระวังเวลา ios, andriod ไม่ได้สำเร็จทั้งคู่ เพราะอาจจะทำให้ version ซํ้ากันได้ (ถ้าrerun workflow) ในเคส production -> ให้ลบอันที่เสร็จข้างเดียวก่อน ควรสร้างใหม่
2. สำหรับ production android บางครั้งมันก็บัค review reject แอพ ให้ไปกดส่ง review อีกรอบใน playstore ก็จะไม่มีปัญหาอะไร
3. ระวังพวก token หมดอายุ เช่นพวก github token,cert แต่ถ้ามันหมดอายุก็สามารถ gen ใหม่แล้วเอามาใส่ได้เลย
4. แต่ถ้า cert หมดอายุ อันนี้ต้องไป gen จาก match ใหม่ โดยให้ใช้คำสั่ง `fastlane match nuke <development หรือ appstore>` ก่อน ค่อน get มันขึ้นมาใหม่ด้วยคำสั่ง `fastlane match <development หรือ appstore> `
5. ในกรณีที่ไม่อยาก build ผ่าน workflow ก็ทำได้สองวิธี
   **_ ถ้า build วิธีนี้ ระวังเรื่องเลข version name มันนำ version ที่ได้จาก semantic release แล้วทีนี้มันอาจจะมีปัญหาตอนอัพขึ้น store_**
   1. Build สดๆ -> กำหนด version name & num ที่ pubspec.yaml ได้เลย แล้ว build ผ่าน คำสั่ง
      **ios**
      `flutter build ipa` สำหรับ ios staging, prod
      แต่แนะนำให้ build ผ่าน xcode แล้วเลือกเมนู product -> archive แล้วเลือก platform ที่จะลงมากกว่า build ผ่าน command มากกว่า

      **android**
      `flutter build apk` สำหรับ android staging
      `flutter build appbundle` สำหรับ android prod
      แล้วเอาไฟล์ทีไ่ด้ไปโยนใน firebase, store
      อย่าลืมเอาไฟล์ key.properties, keystore ไว้ใน โฟลเดอร์ android ด้วย

   2. Build ผ่าน fastlane -> อันนี้ต้อง setup อันนี้ setup ยากนิงนึงแต่ deploy ง่ายกว่า
      ให้เอาพวก secrets ที่ใช้ใน github action มาสร้างเป็นไฟล์ `.env.default` แล้วเอาไว้ในโฟลเดอร์ fastlane แต่ละ platform แล้วเรียกใช้ lane ได้เลย เช่น
      `bundler exec fastlane staging`
      โดยทั้งสองวิธี version name & build จะขึ้นอยู่กับ pubspec.yaml เหมือนกัน
6. ส่วนใหญ่ `MATCH_GIT_BASIC_AUTHORIZATION` ชอบมีปัญหาตอน clone cert ดังนั้นอย่าลืมเช็คว่า token นี้มีสิทธิ์เข้าถึง repo cert ด้วยมั้ย (**อย่าลืม encode base64 ตามคำสั่งด้านบนก่อนใส่ใน secrets**)

**_Matchfile_**
สำคัญมาก ถ้าจะใช้ ci/cd เพราะเป็นตัวทำหน้าที่แทนการ input ของเราตอนใช้คำสั่ง

_IOS only_

```
git_url("https://github.com/LikeMeX/certificate-mobile-ios")



storage_mode("git")



type("appstore") # The default type, can be: appstore, adhoc, enterprise or development



app_identifier("com.likemex.business.futureskill")

username("sivakorn.w@likemeasia.com") # Your Apple Developer Portal username

# For all available options run `fastlane match --help`

# Remove the # in the beginning of the line to enable the other options



# The docs are available on https://docs.fastlane.tools/actions/match
```

หลักๆแล้วก็ให้เปลี่ยน username ให้เป็น account ที่สามารถ access apple dev portal ได้ (ยังไม่ได้ลอง role ไหน แต่พยายามเป็นพวก app manager ขึ้นไปจะเซฟสุด) แต่จริงๆลบไฟล์นี้ไปแล้วค่อยใช้คำสั่งที่บอกด้านล่างนี้

```
cd ./ios
fastlane match init
```

เพื่อตั้งค่า username ที่เราจะใช้แบบนี้ก็ได้เหมือนกัน

---

**APPFILE**
สำคัญเหมือน **MATCHFILE** เลย เพราะเป็นตัวทำหน้าที่แทนการ input ของเราตอนใช้คำสั่ง

_IOS_

```
apple_id("sivakorn.w@likemeasia.com") # Your Apple Developer Portal username



itc_team_id("126673150") # App Store Connect Team ID

team_id("5U99N8R5A7") # Developer Portal Team ID



# For more information about the Appfile, see:

# https://docs.fastlane.tools/advanced/#appfile
```

ตรงนี้เปลี่ยน apple_id ตรงนี้ตรงๆได้เลยไม่มีอะไร แต่ถ้าอยากเปลี่ยนทั้งไฟล์ให้ลบไฟล์นี้ทิ้งแล้วใช้คำสั่ง

```
cd ./ios
fastlane init
```

เพื่อตั้งค่า username ที่เราจะใช้ (มี 2FA ด้วย) ดังนั้น set ผ่านตรงนี้ work สุด เพราะมัน fetch `team_id` กับ `itc_team_id` ได้เลย

_ANDROID_

```
json_key_file("../GHA_PUBLISH_JSON.json") # Path to the json secret file - Follow https://docs.fastlane.tools/actions/supply/#setup to get one

package_name("com.likemex.business.futureskill") # e.g. com.krausefx.app
```

ตรง json_key_file จะเป็นตัว set key ที่ใช้ในการ access เข้าไปที่่ play store ดังนั้นอย่าไปยุ่งกับมันไม่งั้น ci/cd จะมีปัญหาได้

---

\***\*Staging\*\***

เริ่มจากให้ workflow นี้ trigger จากการ push branch `main`

**_`call-workflow-lastest-buildnumber-release`_**

```
create-required-file
```

จะสร้าง file ที่จำเป็นสำหรับการ access เข้าไปที่ platform เช่น key,json รวมถึงพวก env, keystore เพื่อใช้ในการ build app เก็บไว้ใน artifact เพื่อส่งให้ jobs อื่น

```
fastlane-buildnumbers:
```

execute fastlane lane `get_latest_build_number`เพื่อเอา Build Number จากทุก platform
และ `compare-build-number` เพื่อ compare build number ของ ios, android และ return ค่าที่มากที่สุดออกมาที่ `GITHUB_OUTPUT`

**_`increase-lastest-tag`_**

อันนี้ไม่มีอะไร แค่ `BUILD_NUMBER + 1` เพื่อให้เป็น `BUILD_NUMBER` ล่าสุด

**_`call-workflow-semantic-release`_**

call workflow สร้าง tag ใน github tag โดยอิงจาก `git-cz` ประเภท `feature,fix` เพื่อสร้าง tag version -> เราจะเอาตรงนี้แหละ เป็น version name ของ app

**_`deploy-ios-staging`_**

อันนี้สั่ง build ios เพื่อ deploy บน firebase

**_`deploy-andriod-staging`_**

อันนี้สั่ง build android เพื่อ deploy บน firebase

\***\*Production\*\***

เหมือนกัน staging แค่เอา increase number ออก กับ ใช้ tag ที่ release มาใช้ในการเป็น version name ของแอพ
กับ fastlane file ต่างกันนิดหน่อย -> ใน staging จะ build แค่เอาไฟล์ และค่อยเอาไฟล์นั้นใส่ firebase ได้เลย
ส่วน prod คือ build + upload ขึ้น store เลยตรงๆ

**External Docs**

1. Sementic release
   [geek](https://www.geeksforgeeks.org/introduction-semantic-versioning/)
   [medium](https://goangle.medium.com/semantic-versioning-101-ce77a9ba9815)
   [medium](https://medium.com/@pratya.yeekhaday/maven-%E0%B8%81%E0%B8%B2%E0%B8%A3%E0%B8%81%E0%B8%B3%E0%B8%AB%E0%B8%99%E0%B8%94%E0%B9%80%E0%B8%A5%E0%B8%82%E0%B9%80%E0%B8%A7%E0%B8%AD%E0%B8%A3%E0%B9%8C%E0%B8%8A%E0%B8%B1%E0%B8%99%E0%B8%95%E0%B8%B2%E0%B8%A1-semantic-versioning-1776f4e39969)
   [npmjs](https://docs.npmjs.com/about-semantic-versioning)
   [github action](https://github.com/marketplace/actions/action-for-semantic-release)
2. Fetch Coding Number
   1. Android
      [Lastest Google Play Version Code](https://lukebrandonfarrell.medium.com/getting-the-latest-google-play-version-code-with-fastlane-79d809f436d9) but need to do [[Setting Up a Service Account on Google Cloud for Android App Deployment]]
   2. IOS
      [Fetch using fastlane](https://github.com/fastlane/fastlane/discussions/20349)
   3. Firebase Distribution
      [Distribute iOS apps to testers using fastlane](https://firebase.google.com/docs/app-distribution/ios/distribute-fastlane)

Flutter fastlane
https://docs.flutter.dev/deployment/cd

1. Setup Ios
   [[https://medium.com/@manoelsrs/setting-up-a-ci-cd-pipeline-for-ios-using-fastlane-and-github-actions-in-a-flutter-project-8fd350237c33]]
2. Setup Android
   [[https://akexorcist.dev/publish-android-app-to-google-play-with-github-actions/]]

Notification

1. self implement
   [https://github.com/petpusin/github-reuse-workflows/blob/main/.github/workflows/discord-noti.yaml](https://github.com/petpusin/github-reuse-workflows/blob/main/.github/workflows/discord-noti.yaml "https://github.com/petpusin/github-reuse-workflows/blob/main/.github/workflows/discord-noti.yaml")

2. github action
   [https://github.com/marketplace/actions/actions-for-discord](https://github.com/marketplace/actions/actions-for-discord "https://github.com/marketplace/actions/actions-for-discord")

3. Token
   [https://discord.com/api/webhooks/1302889832019656716/rHO7EXxXR5-sos82e7bc2Rypi-e6tT9hiCI\_\_jbEDW8zWK58_ppgrI8IGixoFKhgFbiT](https://discord.com/api/webhooks/1302889832019656716/rHO7EXxXR5-sos82e7bc2Rypi-e6tT9hiCI__jbEDW8zWK58_ppgrI8IGixoFKhgFbiT "https://discord.com/api/webhooks/1302889832019656716/rHO7EXxXR5-sos82e7bc2Rypi-e6tT9hiCI__jbEDW8zWK58_ppgrI8IGixoFKhgFbiT")
