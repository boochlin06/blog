# Retrofit 2 — How to Upload Files and Parameter list to Server

> 原文網址：https://boochlin.com/?p=247
> 發布日期：2016-12-30
> 文章編號：247

---

Retrofit 2

一個久聞大名的網路操作框架，基於 OKHTTP3 在包裝，用起來挺舒爽的，之前都是使用 VOLLEY 作為主要工具，各有優缺點，封裝概念不同。對於 Retrofit 上傳檔案不熟，因此僅僅是作筆記。

[一次上傳一份檔案](https://futurestud.io/tutorials/retrofit-2-how-to-upload-files-to-server)

[一次上傳多份檔案](https://futurestud.io/tutorials/retrofit-2-how-to-upload-multiple-files-to-server)

[同時上傳檔案跟參數](https://futurestud.io/tutorials/retrofit-2-passing-multiple-parts-along-a-file-with-partmap)

其實上面就已經足夠充分，而我的需求則是同時在加上 string list 參數上傳

retrofit2 註解封裝

```
@Multipart
        @POST("/poll")
        Call createVote(@PartMap Map<String, RequestBody> parametor,
                                  @Part("description") RequestBody description,
                                  @Part MultipartBody.Part file

        );
```

Android client code

```
public void createVote(VoteData voteSetting, List options, File image
            , VoteDataManager.createVoteResponseCallback callback) {


        Map<String, RequestBody> parameter = new HashMap<>();

        RequestBody title = RequestBody.create(MediaType.parse("text/plain"), voteSetting.getTitle());
        RequestBody maxOption = RequestBody.create(MediaType.parse("text/plain"), String.valueOf(voteSetting.getMaxOption()));
        RequestBody minOption = RequestBody.create(MediaType.parse("text/plain"), String.valueOf(voteSetting.getMinOption()));
        RequestBody userCanAddOption = RequestBody.create(MediaType.parse("text/plain")
                , String.valueOf(voteSetting.getIsUserCanAddOption()));
        RequestBody userPanPreviewResult = RequestBody.create(MediaType.parse("text/plain")
                , String.valueOf(voteSetting.getIsCanPreviewResult()));
        RequestBody security = RequestBody.create(MediaType.parse("text/plain")
                , VoteData.SECURITY_PUBLIC);
        RequestBody category = RequestBody.create(MediaType.parse("text/plain"), String.valueOf(voteSetting.getCategory()));
        RequestBody startTime = RequestBody.create(MediaType.parse("text/plain"), String.valueOf(voteSetting.getStartTime()));
        RequestBody endTime = RequestBody.create(MediaType.parse("text/plain"), String.valueOf(voteSetting.getEndTime()));

        RequestBody rbOption;
        // STRING LIST PARAMETOR
        for (int i = 0; i < options.size(); i++) {
            rbOption = RequestBody.create(MediaType.parse("text/plain"), options.get(i));
            parameter.put("pt[" + i + "]", rbOption);
        }

        parameter.put("t", title);
        parameter.put("max", maxOption);
        parameter.put("min", minOption);
        parameter.put("add", userCanAddOption);
        parameter.put("res", userPanPreviewResult);
        parameter.put("sec", security);
        parameter.put("cat", category);
        parameter.put("on", startTime);
        parameter.put("off", endTime);

        if (voteSetting.getIsNeedPassword()) {
            RequestBody password = RequestBody.create(MediaType.parse("text/plain"), voteSetting.password);
            parameter.put("p", password);
        }

        RequestBody requestFile = null;
        MultipartBody.Part body = null;
        String descriptionString = "vote_image";
        RequestBody description = null;

        if (image != null) {
            requestFile = RequestBody.create(MediaType.parse("multipart/form-data"), image);
            body = MultipartBody.Part.createFormData("i", image.getName(), requestFile);
            description = RequestBody.create(
                    MediaType.parse("multipart/form-data"), descriptionString);
        }

        Call call = voteService.createVote(parameter, description, body);
        call.enqueue(callback);
    }
```

雖然上面這樣寫好像很蠢，但一時間也找不到怎某寫，

如果有興趣的人，應該要更加進一步理解 multipart/form-data

最近實在是太忙，文章質量提昇不起來
