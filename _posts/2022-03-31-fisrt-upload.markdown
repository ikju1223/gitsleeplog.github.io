---
layout: post
title:  first upload
date:  2022-03-31
image:  02.jpg
tags:   Resources
---
function goRowSpan(tableId, className){      
      var tableRows = $('#'+tableId+" tbody tr");
      var rowEndIdx = [];
      
      $('.'+className).prev('td').each(function() {
    	  debugger;
          rowEndIdx[rowEndIdx.length] = $(this).parent().index();
      });
      if (rowEndIdx.length == 0) rowEndIdx[0] = 0;
      
      for (var i = 0; i < rowEndIdx.length; i++) {            
         var start = rowEndIdx[i];
         var end;
         if (i == rowEndIdx.length-1) {            
            end = tableRows.length;
         }else{
            end = rowEndIdx[i+1];
         }
         for (var j = start; j < end; j++) {
            tableRows.eq(j).children('.'+className).addClass('targets');
         }
         $(".targets").each(function() {
             var rows = $(".targets:contains('" + $(this).text() + "')");
              if (rows.length > 1) {
                 rows.eq(0).attr("rowspan", rows.length);
                 rows.not(":eq(0)").remove();
              }              
         });
         $('.targets').removeClass('targets');
         
      }
      
   }
   

   
   
   /* 테이블 로우 병합(테이블아이디, 병합할 컬럼수 > 왼쪽부터)*/
      function rowMerge(tableName, colums){
         let setTable = $("#"+tableName+" > tbody");
         let totalColums = setTable.find('tr:eq(0)').find('td').length;
         let totalRow = setTable.find('tr').length;
         for(let k = 0; k < totalRow; k++){                                       // 시작 로우 부터 반복
            for(let j = 0; j < colums; j++){                                    // 컬럼 개수 만큼
               let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
               //console.log(thisArea.text());         // 실제 값
               let thisTdCount = thisArea.parent().find('td').length;            
               if(j == (thisTdCount - totalColums + colums)){               
                  break;
               }
               // console.log(k + " " + j);         // k = 행 , 반복되는 열
               let thisTd = thisArea.closest('td').index();                        // 비교될 부분
               let thisTr = thisArea.parent().closest('tr').index();            
               let thisText = thisArea.text();
               
               for(let i = 1; i < (totalRow + 1); i++){            
                  let nextTr = thisArea.parent().parent().find('tr:eq('+(thisTr+i)+')');   //비교할 부분
                  let nextTdCount = nextTr.find('td').length;
                  let nextTdNumber = thisTd - thisTdCount + nextTr.find('td').length;
                  let nextTd = nextTr.find('td:eq('+nextTdNumber+')');
                  let nextText = nextTd.text();            
                  
                  if((nextText != thisText) || (nextTdNumber != 0)){                  // 다르거나,
                     //console.log("("+thisText+")다른 항목입니다 : " + nextText);            // 비교할 영역의 상위 dept가 합쳐져있지 않은경우
                     thisArea.attr("rowspan", i);                              // (비교할 값이 처음으로 오지 않은경우)
                     break;
                  }else{
                     //console.log("("+thisText+")같은 항목입니다 : " + nextText);
                     nextTd.remove();
                  }   
               }
            }                  
         }
      }
   
   
   
   
      function rowMerge2(tableName, colums){
            let setTable = $("#"+tableName+" > tbody");                                 // 테이블 호출
            //let totalColums = setTable.find('tr:eq(0)').find('td').length;                  // 전체 열 갯수
            let totalRow = setTable.find('tr').length;                                 // 전체 행 갯수
            let sameCode = 1;
            for(let k = 0; k < totalRow; k++){                                       // 시작 로우(열) 부터 반복
               for(let j = 0; j < colums; j++){
                  let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
                  let thisText = thisArea.text();
                  
                  // 삭제될 부분은 무시
                  if(thisArea.attr("class") == "deleteCode"){
                     continue;
                  }      
                  
                  // 비교 시작
                  for(let i = 1; i < (totalRow + 1 - k); i++){   
                     let nextArea = setTable.find('tr:eq('+ (k + i) +')').find('td:eq('+ j +')');
                     let nextText = nextArea.text();            
                        
                     if((nextText == thisText) && ((j == 0) || (nextArea.prev('td').attr("sameCode") == thisArea.prev('td').attr("sameCode")))){
                        //console.log(thisText + "  " + nextText);   
                        nextArea.attr("sameCode", sameCode);      // 상위 dept 병합여부 판별용
                        nextArea.attr("class", "deleteCode");      // 삭제될 부분 저장
                     }else{
                        //console.log(thisText + "  " + nextText);        
                        thisArea.attr("sameCode", sameCode);      // 다를경우 중단후 rowpan부여, sameCode 값 증가
                        thisArea.attr("rowspan", i);
                        sameCode = sameCode + 1;
                        break;
                     }               
                  }
               }
            }
            $('.deleteCode').remove();   // 삭제
            $("td").removeAttr("samecode");
         }
      
      
      function rowMerge3(tableName, colums){
            let setTable = $("#"+tableName+" > tbody");                                 // 테이블 호출
            let totalColums = setTable.find('tr:eq(0)').find('td').length;                  // 전체 행 갯수
            let totalRow = setTable.find('tr').length;                                 // 전체 열 갯수
            let sameCode = 1;
            if(colums > totalColums){
               colums = totalColums;
            }
            for(let j = 0; j < colums; j++){
               for(let k = 0; k < totalRow; k++){                                    
                  let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
                  let thisText = thisArea.text();
                  
                  // 비교 시작
                  for(let i = 1; i < (totalRow + 1 - k); i++){   
                     let nextArea = setTable.find('tr:eq('+ (k + i) +')').find('td:eq('+ j +')');
                     let nextText = nextArea.text();            
                     
                     // 비교 대상이 같고, 상위 dept가 병합되어있는(혹은 상위 dept가 없는경우) 경우만 같은 요소로 취급
                     if((nextText == thisText) && ((j == 0) || (nextArea.prev('td').attr("sameCode") == thisArea.prev('td').attr("sameCode")))){
                        //console.log(thisText + "  " + nextText);   
                        nextArea.attr("sameCode", sameCode);      // 상위 dept 병합여부 판별용
                        nextArea.attr("class", "deleteCode");      // 삭제될 부분 저장
                     }else{
                        //console.log(thisText + "  " + nextText);
                        thisArea.attr("sameCode", sameCode);      // 다를경우 중단후 rowpan부여, sameCode 값 증가
                        thisArea.attr("rowspan", i);
                        k = k + i - 1;
                        sameCode = sameCode + 1;
                        break;
                     }               
                  }
               }
            }
            $('.deleteCode').remove();   // 삭제
            $("td").removeAttr("samecode");
         }
         
         function rowMerge4(tableName, colums){
            let setTable = $("#"+tableName+" > tbody");
            let sameCode = 1;
            let k = 0; let i = 1; let j = 0;      
            while(true){
               let thisArea = setTable.find('tr:eq('+ k +')').find('td:eq('+ j +')');
               let nextArea = setTable.find('tr:eq('+ (k + i) +')').find('td:eq('+ j +')');
               if((nextArea.text() == thisArea.text()) && ((j == 0) || (nextArea.prev('td').attr("sameCode") == thisArea.prev('td').attr("sameCode")))){
                  nextArea.attr("sameCode", sameCode);      // 상위 dept 병합여부 판별용
                  nextArea.attr("class", "deleteCode");      // 삭제될 부분 저장
                  i++;
               }else{
                  thisArea.attr("sameCode", sameCode);      // 다를경우 중단후 rowpan부여, sameCode 값 증가
                  thisArea.attr("rowspan", i);
                  k = k + i;                        // 삭제될 부분 넘어가기
                  if(k >= (setTable.find('tr').length-1)){
                     k = 0;
                     j++;
                  }
                  if(j >= colums)
                     break;               
                  i = 1;                              // 비교될 부분 다음 값을 초기화
                  sameCode = sameCode + 1;
               }               
            }               
            $('.deleteCode').remove();   // 삭제
            $("td").removeAttr("samecode");
         }
   
   		// table id 정보, 합칠 cloums 목록 필요(ex. [0,1,2,5])
         function rowMergeList(tableName, colum){
   			let colums = [];
   			if(typeof(colum) == 'number'){
   				for(let i = 0; i<colum; i++){
   					colums.push(i);
   				}
   			}else{
   				colums = colum;
   			}
             let setTable = $("#"+tableName+" > tbody");			// 테이블 호출
             let totalRow = setTable.find('tr').length;				// 전체 열 갯수
             let sameCode = 1;										// 상위 dept 병합 여부 판별 번호			
             let PrevTd = 0;										// 이전 Td 위치
             for(const j of colums){
                for(let k = 0; k < totalRow; k++){ 
                   let thisTr = setTable.find('tr:eq('+ k +')')		// 비교될 값
                   let thisArea = thisTr.find('td:eq('+ j +')');
                   let thisText = thisArea.text();                   
                   // 비교 시작
                   for(let i = 1; i < (totalRow + 1 - k); i++){ 
                	  let nextTr = setTable.find('tr:eq('+ (k + i) +')');	// 비교할 값
                      let nextArea = nextTr.find('td:eq('+ j +')');
                      let nextText = nextArea.text();            
                      
                      // 비교 대상이 같고, 상위 dept가 병합되어있는(혹은 상위 dept가 없는경우) 경우만 같은 요소로 취급
                      if((nextText == thisText) 
                    		  && ((PrevTd == 0) 
                    				  || (nextTr.find('td:eq('+ PrevTd +')').attr("sameCode") == thisTr.find('td:eq('+ PrevTd +')').attr("sameCode")))){
                         //console.log(thisText + "  " + nextText);   
                         nextArea.attr("sameCode", sameCode);      // 상위 dept 병합여부 판별용
                         nextArea.attr("class", "deleteCode");      // 삭제될 부분 저장
                      }else{
                         //console.log(thisText + "  " + nextText);
                         thisArea.attr("sameCode", sameCode);      // 다를경우 중단후 rowpan부여, sameCode 값 증가
                         thisArea.attr("rowspan", i);
                         k = k + i - 1;
                         sameCode = sameCode + 1;
                         break;
                      }               
                   }
                }
                PrevTd = j;
             }
             $('.deleteCode').remove();   // 삭제
             $("td").removeAttr("samecode");
          }
         