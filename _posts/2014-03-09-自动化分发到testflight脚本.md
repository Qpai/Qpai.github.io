最近公司项目有一些分发的需求，所以我搜索了一些资料，实现了自动打包并且上传到testflight的脚本。

大概需求有这些

1. 将版本号和用户名显示在图标上面
2. Archive一个HOC的包
3. 本地备份一个ipa和dsym文件
4. 上传至Testflight

一、图标显示版本号
--
1. 安装两个shell工具

        brew install imagemagick  
        brew install ghostscript
        
2. 在Build Phases，新建一个Run Script。

        
           #这里可以讲版本号设置成SVN或者git的版本号，以防冲突
    #/usr/libexec/PlistBuddy -c "Set :CFBundleVersion ${SVN_REVISION}""$INFO_PLIST_PATH"
    version=`/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${INFOPLIST_FILE}"`
    
    function processIcon() {
        export PATH=$PATH:/usr/local/bin
        base_file=$1
        base_path=`find "${SRCROOT}/<project name>" -name $base_file`
        
        echo "Processing 43 $base_path"
        
        if [[ ! -f ${base_path} || -z ${base_path} ]]; then
        return;
        fi
        
        echo " 11 ${target_path}"
        
        target_file=$base_file
        target_path="${CONFIGURATION_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/${target_file}"
        
        
        
        if [ $CONFIGURATION = "Release" ]; then
        cp "${base_path}" "$target_path"
        return
        fi
        
        width=`identify -format %w "${base_path}"`
        
        convert -background '#0008' -fill white -gravity center -size ${width}x40\
        caption:"${version} ${LOGNAME}"\
        "${base_path}" +swap -gravity south -composite "${target_path}"
        
        echo "Overlayed ${target_path}"
    }
    
    processIcon "Icon_base.png"
    processIcon "Icon_base@2x.png"
            

* 这里只涉及到了Icon.png和Icon@2x.png的图标，还可以增加更多。  
* 处理了release的配置，如果发布状态，则不显示版本号，可以根据需求修改。  
* 脚本编辑框下面有一个选项：Show environment variables in build   log。如果勾上，那你的log就会出现一堆本地的环境变量及其值，你可以打印出来，写脚本的时候就知道哪些是可以用的环境变量。但是如果调试shell脚本的时候最好不勾上，因为很多，而log最多显示200行，你shell的log可能会被隐藏掉，影响你的调试。  


二、自动打包和上传。
----
在xcode左上角找到edit schame，然后找到左边有Archive，点击旁边的三角，在post-actions增加一个run script。因为需要archive之后，才能找到相应的xcarchive文件，所以是后置运行脚本。



shell脚本如下：
    #!/bin/sh
    
    #  Script.sh
    #  IconOverlaying
    #
    #  Created by Li Yi on 14-3-9.
    #  Copyright (c) 2014年 pixle. All rights reserved.
    
    
    # define vars
    TESTFLIGHT_TEAM_TOKEN="<team token>"
    TESTFLIGHT_API_TOKEN="<user token>"
    TESTFLIGHT_DISTRIBUTION_LIST="QpaiTeam"
    BACK_UP_PATH="${SOURCE_ROOT}/IpaBackup"
    
    
    
    echo "find last Archives"
    newest=
    backIFS=$IFS
    IFS=$(echo -en '\n\b')
    for f in `find ~/Library/Developer/Xcode/Archives -name *.xcarchive`
    do
    if [ -z $newest ]
    then
    newest=$f
    elif [ $f -nt $newest ]
    then
    newest=$f
    fi
    done
    echo "${f}"
    FS=$backIFS
    
    LASTARCHIVE_PATH=$f
    
    # it's build version,or git version or svn version
    VERSION=`/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${INFOPLIST_FILE}"`
    
    IPA_PATH="${BACK_UP_PATH}/${VERSION}.ipa"
    DSYM_PATH="${LASTARCHIVE_PATH}/dSYMs/${PROJECT_NAME}.app.dSYM"
    COPY_DSYM_PATH="${BACK_UP_PATH}/${VERSION}.app.dSYM"
    
    # create backup folder
    mkdir -p "${BACK_UP_PATH}"
    
    rm "${IPA_PATH}"
    
    echo "create ipa from archive"
    
    xcodebuild -exportArchive -exportFormat IPA -archivePath "${LASTARCHIVE_PATH}" -exportPath "${IPA_PATH}"
    
    echo "copy dsym to back-up"
    cp -r "${DSYM_PATH}" "${COPY_DSYM_PATH}"
    
    
    #zipping the .dSYM to send to Testflight
    echo "Generating dsym zip file"
    /usr/bin/zip -r "${COPY_DSYM_PATH}.zip""${COPY_DSYM_PATH}"
    
    # sends the .ipa file to TestFlight
    
    
    
    
    echo "Sending to TestFlight"
    curl http://testflightapp.com/api/builds.json -F file="@${IPA_PATH}" \
    -F dsym="@${COPY_DSYM_PATH}.zip" \
    -F api_token="${TESTFLIGHT_API_TOKEN}" \
    -F team_token="${TESTFLIGHT_TEAM_TOKEN}" \
    -F notes="This build was uploaded via the upload API" \
    -F notify=False \
    -F distribution_lists="${TESTFLIGHT_DISTRIBUTION_LIST}"
    echo Submission ended

###执行了如下功能：

1. 寻找最后一个archive
2. 生成一个本地备份的文件夹
3. 通过最后一个archive生成一个以build版本号命名的ipa，并且保存于备份文件夹中。
4. 把最后一个archive的dsym文件复制到备份文件中，并且重命名为版本号。
5. 把dsym文件打包成zip
6. 通过curl将ipa和dysm上传到testflight

###解释一些需要设置的变量名含义：


1. TESTFLIGHT_TEAM_TOKEN就是你Testflight里面team的token
2. TESTFLIGHT_API_TOKEN是你用户的token
3. TESTFLIGHT_DISTRIBUTION_LIST是你的包分发的目标受众，可以是team名，也可以是用户名，用逗号隔开。



注意： 最下面的curl就是上传到testflight的接口，和传参。具体是不是上传正确了，需要看log。