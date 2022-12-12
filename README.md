# Mglist

Entity
----------
namespace Domain.CallCenter.MsgList.Entity;

public record msgListFindByOperatorIdAndCallDate
{
    public string? seqNo { get; init; }
    public string? tokCd { get; init; }
    public string? callType { get; init; }
    public string? callTypeNm { get; init; }
    public string? resultFlg { get; init; }
    public string? callDt { get; init; }
    public string? callHour { get; init; }
    public string? callMin { get; init; }
    public string? callClass { get; init; }
    public string? callClassNm { get; init; }
    public string? memo { get; init; }
    public string? urgentFlg { get; init; }
    public string? custNm { get; init; }
    public string? custNmKn { get; init; }
    public string? inpTanCd { get; init; }
    public string? inpTanNm { get; init; }
    public string? destTanCd { get; init; }
    public string? destTanNm { get; init; }
    public string? datFlg { get; init; }
    public string? crtDt { get; init; }
    public string? crtTm { get; init; }
    public string? wrtDt { get; init; }
    public string? wrtTm { get; init; }
}

\\\\\\\\\---repository
--------------
using Domain.CallCenter.MsgList.Entity;

namespace Domain.CallCenter.MsgList.Repository
{
    public interface IMsgList
    {
        IEnumerable<msgListFindByOperatorIdAndCallDate> findByOperatorIdAndCallDate(string oreratorId, string callDate);
    }
}
\\\-----service1
---
namespace Domain.CallCenter.MsgList.Service
{
    public interface IMsgListService
    {
        string urgentToModel(string value);

        bool urgentFlgToModel(string value);

        string resultToModel(string value);

        bool resultFlgToModel(string value);

        bool datFlgToModel(string value);

        bool defaultFlgToModel(string value);
    }
}
-----service 2
----
using System.Collections.ObjectModel;
using Infrastructure.Constants;

namespace Domain.CallCenter.MsgList.Service
{
    public class msgListService : IMsgListService
    {
        private const string CST_STATUS_COMP = "完了";

        private const string CST_CODE_NOCOMP = "0";
        private const string CST_CODE_COMP = "1";

        private const string CST_STATUS_URGENT = "1";
        private const string CST_MARKS_URGENT = "●";

        private static readonly ReadOnlyCollection<string> callClassList = Array.AsReadOnly(new string[] {"00001","00006","00013","00014"});

        #region 至急設定
        /// <summary>
        /// 至急設定
        /// </summary>
        /// <param name="value(至急フラグ)"></param>
        /// <returns>至急時：●、以外：""</returns>
        /// <remarks>
        /// 至急フラグが"1"の場合、「●」をセット
        /// </remarks>
        public string urgentToModel(string value)
        {
            return value == CST_STATUS_URGENT ? CST_MARKS_URGENT : "";
        }
        #endregion

        #region 至急（フラグ）設定
        /// <summary>
        /// 至急（フラグ）設定
        /// </summary>
        /// <param name="value(至急フラグ)"></param>
        /// <returns>true(完了) / false(未)</returns>
        /// <remarks>
        /// 至急をbool値で返却する
        /// </remarks>
        public bool urgentFlgToModel(string value)
        {
            return value == CST_STATUS_URGENT ? true : false;
        }
        #endregion

        #region コール結果（状態）設定
        /// <summary>
        /// コール結果（状態）設定
        /// </summary>
        /// <param name="value(コール結果)"></param>
        /// <returns>コール結果=1：完了、コール結果=0：""</returns>
        /// <remarks>
        /// コール結果が"1"の場合、「完了」をセット
        /// </remarks>
        public string resultToModel(string value)
        {
            return value == CST_CODE_COMP ? CST_STATUS_COMP : "";
        }
        #endregion

        #region コール結果（フラグ）設定
        /// <summary>
        /// コール結果（フラグ）設定
        /// </summary>
        /// <param name="value(コール結果)"></param>
        /// <returns>true(完了) / false(未)</returns>
        /// <remarks>
        /// コール結果をbool値で返却する
        /// </remarks>
        public bool resultFlgToModel(string value)
        {
            return value == CST_CODE_COMP ? true : false;
        }
        #endregion

        #region 論理削除フラグ設定
        /// <summary>
        /// 論理削除フラグ設定
        /// </summary>
        /// <param name="value(論理削除フラグ)"></param>
        /// <returns>true(削除) / false(使用中)</returns>
        /// <remarks>
        /// 論理削除フラグをbool値で返却する
        /// </remarks>
        public bool datFlgToModel(string value)
        {
            return value == CommonConst.CST_DATKB.DEL ? true : false;
        }
        #endregion

        #region 初期表示フラグ設定
        /// <summary>
        /// 初期表示フラグ設定
        /// </summary>
        /// <param name="value(コール区分)"></param>
        /// <returns>true(初期表示対象) / false(初期表示対象外)</returns>
        /// <remarks>
        /// 初期表示フラグをbool値で返却する
        /// コール区分が以下の場合、コール区分未選択時の初期表示データとして表示対象とする
        /// 　00001：伝言
        /// 　00006：返品理由保留
        /// 　00013：返品確認
        /// 　00014：調査依頼
        /// </remarks>
        public bool defaultFlgToModel(string value)
        {
            return callClassList.Contains(value) ? true : false;
        }
        #endregion
    }
}
-----
constroller
-------
using Microsoft.AspNetCore.Mvc;
using Application.CallCenter.MsgList.Responses;
using Domain.CallCenter.MsgList.Repository;
using Domain.CallCenter.CallClass.Repository;

namespace Application.CallCenter.MsgList.Controllers
{
    [Route("api/msgList")]
    [ApiController]
    public class MsgListController : ControllerBase
    {
        private readonly IMsgList _msgList;
        private readonly ICallClass _callClass;

        public MsgListController(IMsgList msgList, ICallClass callClass)
        {
            _msgList = msgList;
            _callClass = callClass;
        }

        #region 伝言状況情報取得
        /// <summary>
        /// 伝言状況情報取得
        /// 使用画面：伝言状況
        /// </summary>
        /// <param name="operatorId">担当者コード（自動送信分の場合は99999999）</param>
        /// <param name="mode">コール日付</param>
        /// <returns></returns>
        /// <remarks>伝言状況情報を取得します。</remarks>
        [HttpGet("detail")]
        public ActionResult<IEnumerable<msgListResponse>> findByOperatorIdAndCallDate([FromQuery] string operatorId, [FromQuery] DateTime callDate)
        {
            try
            {
                var rtn = _msgList.findByOperatorIdAndCallDate(operatorId, callDate.ToString("yyyyMMdd"));
                return rtn == null ? null : msgListResponse.ToResponse(rtn).ToList();
            }
            catch (Exception ex)
            {
                return StatusCode(500, ex.Message);
            }
        }
        #endregion

        #region コール区分コンボボックス情報取得
        /// <summary>
        /// コール区分コンボボックス情報取得
        /// </summary>
        /// <returns></returns>
        /// <remarks>コール区分（23）のキーコードと名称１を取得します。</remarks>
        [HttpGet("callClass")]
        public ActionResult<IEnumerable<callClassResponseToMsgList>> findCdAndNm()
        {
            try
            {
                var rtn = _callClass.findCdAndNmForCallClass();
                return rtn == null ? null : callClassResponseToMsgList.ToResponse(rtn).ToList();
            }
            catch (Exception ex)
            {
                return StatusCode(500, ex.Message);
            }
        }
        #endregion
    }
}
-----CallClassResponse
----
using Domain.CallCenter.CallClass.Entity;

namespace Application.CallCenter.MsgList.Responses
{
    public class callClassResponseToMsgList
    {
        // キーコード
        public string cd1 { get; set; }

        // 名称
        public string nm1 { get; set; }

        public static IEnumerable<callClassResponseToMsgList> ToResponse(IEnumerable<callClass> sourse)
        {
            return from row in sourse
                select new callClassResponseToMsgList()
                {
                    cd1 = row.cd1 ?? string.Empty,
                    nm1 = row.nm1 ?? string.Empty

                };
        }
    }
}
---mglist resopnes
\\\\----
using Domain.CallCenter.MsgList.Entity;
using Domain.CallCenter.MsgList.Service;

namespace Application.CallCenter.MsgList.Responses
{
    public record msgListResponse
    {
        // 送信日時
        public string? sendDt { get; set; }

        // 送信先
        public string? operatorNm { get; set; }

        // 至急
        public string? urgent { get; set; }

        // 顧客名
        public string? customerNm { get; set; }

        // 分類
        public string? callType { get; set; }

        // メモ
        public string? memo { get; set; }

        // 状態
        public string? result { get; set; }

        // 顧客コード
        public string? customerCd { get; set; }

        // 連番
        public string? seqNo { get; set; }

        // 至急フラグ
        public bool? urgentFlg { get; set; }

        // コール結果
        public bool? resultFlg { get; set; }

        // コール区分
        public string? callClass { get; set; }

        // 分類名称（画面表示）
        public string? callTypeTxt { get; set; }

        // 作成日
        public string? crtDt { get; set; }

        // 更新日
        public string? wrtDt { get; set; }

        // コール日付
        public string? callDt { get; set; }

        // コール時
        public string? callHour { get; set; }

        // コール分
        public string? callMin { get; set; }

        // 論理削除フラグ
        public bool? datFlg { get; set; }

        // 初期表示フラグ
        public bool? defaultFlg { get; set; }

        public static IEnumerable<msgListResponse> ToResponse(IEnumerable<msgListFindByOperatorIdAndCallDate> sourse)
        {
            var msgListService = new msgListService();

            return from row in sourse
                   select new msgListResponse()
                   {
                       sendDt = row.crtDt.Substring(0, 4) + "/" + row.crtDt.Substring(4, 2) + "/" + row.crtDt.Substring(6, 2) + " " + row.crtTm.Substring(0, 2) + ":" + row.crtTm.Substring(2, 2),
                       operatorNm = row.destTanNm?? string.Empty,
                       urgent = row.urgentFlg == null ? "" : msgListService.urgentToModel(row.urgentFlg),
                       callTypeTxt = row.callType.Trim() == "" ? row.callClassNm : row.callClassNm + "：" + row.callTypeNm,
                       customerNm = row.custNm.Trim() == "" ? row.custNmKn : row.custNm,
                       memo = row.memo ?? string.Empty,
                       result = row.resultFlg == null ? "" : msgListService.resultToModel(row.resultFlg),
                       customerCd = row.tokCd ?? string.Empty,
                       seqNo = row.seqNo ?? string.Empty,
                       urgentFlg = row.urgentFlg == null ? false : msgListService.urgentFlgToModel(row.urgentFlg),
                       resultFlg = row.resultFlg == null ? false : msgListService.resultFlgToModel(row.resultFlg),
                       callType = row.callType ?? string.Empty,
                       callClass = row.callClass ?? string.Empty,
                       crtDt = row.crtDt ?? string.Empty,
                       wrtDt = row.wrtDt ?? string.Empty,
                       callDt = row.callDt ?? string.Empty,
                       callHour = row.callHour ?? string.Empty,
                       callMin = row.callMin ?? string.Empty,
                       datFlg = row.datFlg == null ? false : msgListService.datFlgToModel(row.datFlg),
                       defaultFlg = row.callClass == null ? false : msgListService.defaultFlgToModel(row.callClass)
                   };
        }
    }
}
