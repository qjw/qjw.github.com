---
layout: post
title:  让cron在某个每天某个时间段随机触发的工具脚本
category: bash
---

##需求
希望能够配置一个时间段（单位：小时），不超过24小时。在这个时间段内能够随机触发一个调用。支持配置诸如23~4点（跨天）这样的时间段。

        #!/bin/bash

        declare g_start_hour=23
        declare g_end_hour=22
        declare g_flag_file="/tmp/test"

        function main()
        {
            # check invalid,不能相等，若相等可以用0~24替代
            if [ "${g_start_hour}" -eq "${g_end_hour}" ];then
                echo "invalid param"
                return 1
            elif [ "${g_start_hour}" -gt "${g_end_hour}" ];then
                # 若start为24，转成0
                if [ "${g_start_hour}" -eq "24" ];then
                    g_start_hour = 0
                fi
                # 若end为0，转成24
                if [ "${g_end_hour}" -eq "0" ];then
                    g_end_hour = 24
                fi
            fi

            # adjust hour，获得当前时间
            local current_hour=$(date +"%k")
            # 若开始时间晚于终止时间，也就是跨度两天，
            if [ "${g_start_hour}" -gt "${g_end_hour}" ];then
                # 若当前时间恰好在跨度的第二天范围内，也将其+24
                if [ "${current_hour}" -lt "${g_end_hour}" ];then
                    current_hour=$((${current_hour} + 24))
                fi
                # 那么将终止时间+24
                g_end_hour=$((${g_end_hour} + 24))
            fi

            echo "hour ${g_start_hour} to ${g_end_hour},current_hour ${current_hour}"
            # is invalid current hour
            # 判断当前时间是否在配置的范围内
            if [ "${current_hour}" -lt "${g_start_hour}" -o \
                "${current_hour}" -ge "${g_end_hour}" ];then
                return 0
            fi

            # 配置的时间范围，总的小时数
            local hour_count=$((${g_end_hour} - ${g_start_hour}))
            echo "hour_count ${hour_count}"

            # 若的当前日期
            local current_date=$(date +"%Y/%m/%d:%H")
            if [ "${current_hour}" -ge "24" ];then
                # 若跨度两天，并且当前时间位于第二天，那么当前日期是前一天
                current_date=$(date -d "1 days ago" +"%Y/%m/%d:%H")
            fi
            echo "current_date ${current_date}"

            # 尝试从一个配置中读取时间
            touch "${g_flag_file}"
            local history_date=$(tail -c 32 "${g_flag_file}")
            echo "history_date ${history_date}"
            # 若相等表示，当日已经处理
            if [ ! -z "${history_date}" -a \
                "${current_date}" = "${history_date}" ];then
                echo "has updated"
                return 0
            else
                # 否则清空
                echo -n "" > "${g_flag_file}"
            fi

            # 获得一个0 <= x < hour_count范围内的随机数
            # 也就是配置的这hour_count小时的概率相等
            local update_flag=$(($RANDOM % ${hour_count}))
            echo "update_flag ${update_flag}"
            # 若等于0，这表示触发事件
            # 同时，若一直都没触发，到了配置时间范围的最后一个小时，那么强制触发
            current_hour=$((${current_hour} + 1))
            if [ "${update_flag}" -eq "0" -o \
                "${current_hour}" -eq "${g_end_hour}" ];then
                # 将日期写道配置以作记录
                echo -n "${current_date}" > "${g_flag_file}"
                echo "start to update"
                return 0
            fi
        }

        main "${@}"

